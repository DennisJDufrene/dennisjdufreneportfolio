# Stale Content Alert

Documentation decays. APIs change, procedures get updated, and product details shift.
However, the documentation often does not get updated with these changes. In most workflows, 
stale content is only discovered when a reader notices something is wrong. By then the 
damage is already done.

This workflow moves stale content detection upstream. It monitors the `docs` directory,
identifies files that have not been modified in over seven days, and automatically opens
a labeled GitHub issue listing each file that needs review. Content governance becomes
a system behavior rather than a calendar reminder.

## The Problem It Solves

Content review processes typically rely on writers remembering to audit their own work.
That works at small scale. It breaks down as a documentation set grows, contributors
multiply, or project velocity increases.

A seven-day threshold is intentionally short for a portfolio context — it surfaces the
mechanism clearly. In a production documentation set the threshold would typically be
30, 60, or 90 days depending on how frequently the underlying product changes. The
logic is identical regardless of the interval.

## How It Works

The workflow checks out the full Git history and loops through every file in the `docs`
directory. For each file it retrieves the timestamp of the most recent commit that touched
it. Any file not committed within the threshold window is added to a list.

If stale files are found, the workflow calls the GitHub Issues API and opens a single
issue titled **"Old Files Detected in Docs Directory"** with each stale file listed as
a bullet point under the `stale` label. If no stale files are found, the workflow exits
cleanly with no side effects.

## Workflow File

```yaml
name: Check Date Modified

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  check-old-files:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

      - name: Find old files in docs directory
        id: find_old_files
        run: |
          seven_days_ago=$(date -d '7 days ago' +%s)
          old_files=()

          while IFS= read -r -d '' file; do
            modified_time=$(git log -1 --format="%at" -- "$file")
            if [ "$modified_time" -lt "$seven_days_ago" ]; then
              old_files+=("$file")
            fi
          done < <(find docs/ -type f -print0)

          if [ ${#old_files[@]} -gt 0 ]; then
            echo "Old files found in docs directory and its subdirectories:"
            printf '%s\n' "${old_files[@]}"
            printf '%s\n' "${old_files[@]}" > old_files.txt
          else
            echo "No old files found in docs directory."
            echo "" > old_files.txt
          fi

      - name: Set OLD_FILES environment variable
        id: set_old_files
        run: |
          OLD_FILES=$(cat old_files.txt)
          echo "OLD_FILES<<EOF" >> $GITHUB_ENV
          echo "$OLD_FILES" >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV

      - name: Create GitHub Issue
        if: env.OLD_FILES != ''
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          title="Old Files Detected in Docs Directory"
          body="The following files in the \`docs\` directory and its subdirectories are older than 7 days:\n\n"
          body+=$(echo "${{ env.OLD_FILES }}" | sed 's/^/* /')

          json=$(jq -n --arg title "$title" --arg body "$body" \
            '{title: $title, body: $body, labels: ["stale"]}')

          curl -X POST \
            -H "Authorization: token $GITHUB_TOKEN" \
            -H "Accept: application/vnd.github.v3+json" \
            -d "$json" \
            https://api.github.com/repos/${{ github.repository }}/issues
```

## Design Decisions

**Git history as the source of truth**

The workflow uses `git log -1 --format="%at"` to determine when a file was last modified
rather than the filesystem's `mtime`. Filesystem timestamps are unreliable in CI environments
— a fresh checkout resets them. Git commit timestamps reflect when a human actually changed
the content, which is the meaningful signal for a content review workflow.

This requires `fetch-depth: 0` on checkout. A shallow clone would only pull recent history
and return empty timestamps for files that haven't changed recently, producing false positives.
Fetching full history ensures every file has a resolvable commit timestamp.

**Single consolidated issue per run**

Rather than opening one issue per stale file, the workflow batches all findings into a single
issue. This keeps the GitHub Issues queue clean and makes the alert actionable — a writer
opens one issue, works through the list, and closes it when done.

**`stale` label for triage**

The issue is created with a `stale` label automatically. In a team workflow this allows
issues to be filtered, assigned, and tracked separately from feature work or bug reports.
The label must exist in the repository before the workflow runs.

**`workflow_dispatch` trigger**

In addition to running on push to `main`, the workflow can be triggered manually from
the GitHub Actions tab via `workflow_dispatch`. This is useful for running an audit on
demand without needing to commit a change — for example, before a scheduled content review
or after a major product update.

**Conditional issue creation**

The `Create GitHub Issue` step only runs when `OLD_FILES` is non-empty. If all files in
the `docs` directory are current, the workflow completes successfully with no issue opened
and no noise in the repository.

**Authenticated API call with scoped token**

The workflow uses `secrets.GITHUB_TOKEN`, which GitHub automatically provides for every
workflow run. It is scoped to the repository and expires when the run ends. No personal
access token or stored credential is required.

## Trigger and Permissions

| Trigger | Behavior |
|---|---|
| `push` to `main` | Runs automatically on every merge |
| `workflow_dispatch` | Runs on demand from the Actions tab |

No explicit `permissions` block is required because `GITHUB_TOKEN` has issues write access
by default for public repositories. For private repositories or organizations with restricted
default permissions, adding `permissions: issues: write` to the workflow is recommended.

## Adapting the Threshold

The seven-day window is set in a single line and easy to adjust:

```bash
seven_days_ago=$(date -d '7 days ago' +%s)
```

Replace `7 days ago` with any interval recognized by the GNU `date` command:

| Threshold | Value |
|---|---|
| 30 days | `30 days ago` |
| 90 days | `90 days ago` |
| 6 months | `6 months ago` |
| 1 year | `1 year ago` |

In a production documentation set, the right threshold depends on how frequently the
underlying product changes. Fast-moving APIs might warrant 30 days. Stable reference
documentation might use 6 months or longer.

## About This Sample

This workflow represents a content governance approach that is often discussed in
documentation strategy but rarely implemented mechanically. The usual alternative is
a spreadsheet, a recurring calendar event, or nothing at all.

Encoding the review trigger as an automated workflow means it runs consistently regardless
of team size, workload, or institutional memory. It is included here as a demonstration
that documentation quality can be enforced at the infrastructure level, not just through
style guides and good intentions.
