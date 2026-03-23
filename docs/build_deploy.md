# Site Build and Deploy

This workflow automates the build and deployment of this portfolio site on every push to the
`main` branch. It eliminates manual publishing steps, ensures the live site always reflects
the current state of the repository, and enforces a Docs-as-Code principle: documentation
is treated like software, with source control and automated delivery as first-class concerns.

## The Problem It Solves

Manual publishing workflows introduce lag and human error. A writer finishes an edit, commits
it, then has to remember to run a build command and push the output separately. In a team
context that compounds quickly — multiple contributors, inconsistent local environments, and
no guarantee that what ships matches what was reviewed.

This workflow removes that entirely. A merged pull request is the publish action. Everything
downstream is automated.

## How It Works

The workflow triggers on any push to `main` and runs on a GitHub-hosted Ubuntu runner.
It installs Python, restores a dependency cache, installs the MkDocs Material theme and
required plugins, then deploys the built site to the `gh-pages` branch using
`mkdocs gh-deploy`.

GitHub Pages serves the `gh-pages` branch directly, so the site is live within seconds
of the workflow completing.

## Workflow File

```yaml
name: publish 
on:
  push:
    branches:      
      - main
permissions:
  contents: write
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Configure Git Credentials
        run: |
          git config user.name github-actions[bot]
          git config user.email 41898282+github-actions[bot]@users.noreply.github.com
      - uses: actions/setup-python@v5
        with:
          python-version: 3.x
      - run: echo "cache_id=$(date --utc '+%V')" >> $GITHUB_ENV 
      - uses: actions/cache@v4
        with:
          key: mkdocs-material-${{ env.cache_id }}
          path: .cache
          restore-keys: |
            mkdocs-material-
      - run: pip install mkdocs-material
      - run: pip install mkdocs-include-markdown-plugin
      - run: pip install mkdocs-table-reader-plugin
      - run: pip install mkdocs-img2fig-plugin
      - run: pip install mkdocs-macros-plugin
      - run: pip install mkdocs-render-swagger-plugin
      - run: mkdocs gh-deploy --force
```

## Design Decisions

**Weekly dependency caching**

The workflow uses a cache keyed to the UTC week number (`%V`). MkDocs Material and its
plugins rarely change week to week, so caching the `.cache` directory avoids redundant
pip installs on every run and cuts build time meaningfully. The cache is refreshed
automatically at the start of each new week, keeping dependencies reasonably current
without a manual update step.

**Bot identity for Git commits**

The `gh-deploy` command commits the built site to the `gh-pages` branch. Configuring
the GitHub Actions bot identity (`github-actions[bot]`) keeps the commit history clean
and makes it immediately clear which commits were machine-generated versus human-authored.

**Plugin inventory**

Each plugin is installed explicitly rather than via a `requirements.txt` file. This is a
deliberate transparency choice for a portfolio context — the list documents exactly what
the site depends on and why:

| Plugin | Purpose |
|---|---|
| `mkdocs-material` | Theme, navigation, search, and admonition support |
| `mkdocs-include-markdown-plugin` | Pulls shared content snippets into multiple pages |
| `mkdocs-table-reader-plugin` | Renders CSV data as formatted tables |
| `mkdocs-img2fig-plugin` | Wraps images in `<figure>` elements with caption support |
| `mkdocs-macros-plugin` | Enables Jinja2 variables and macros across pages |
| `mkdocs-render-swagger-plugin` | Renders OpenAPI specs as interactive Swagger UI |

**`--force` flag on deploy**

The `mkdocs gh-deploy --force` flag overwrites the `gh-pages` branch on every run rather
than attempting a merge. Since the branch contains only build artifacts, force-pushing is
safe and avoids conflicts that could stall a deployment.

## Trigger and Permissions

The workflow runs on `push` to `main` only. Pull request branches do not trigger a deploy,
which means work-in-progress content cannot accidentally reach the live site before review.

The `permissions: contents: write` block grants the workflow the minimum access needed to
commit the built output to `gh-pages`. No other repository permissions are elevated.

## About This Sample

This workflow is pulled directly from the GitHub Actions configuration powering this
portfolio site. It is not a constructed example. The page you are reading right now
was published by this workflow.

It is included here as a demonstration of Docs-as-Code practice: documentation
infrastructure treated with the same rigor as application infrastructure, version-controlled,
automated, and documented.
