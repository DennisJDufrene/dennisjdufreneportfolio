!!! info
    
    This is a sample YAML help file document targeting developers.

# Fake System Report Configuration

The **Fake System Report** is configured with the `configure_report.yaml` file located in `\etc\fake_system\reports\`.

## Required Fields

The following fields must be included in the config file:

```text linenums="1"
#report_template: The report template location and filename used to generate the report.
report_template: "\etc\fake_system\templates\fake_system_report.template"

# header_file: The location and filename of the file used to populate the report header.
header_file: "\etc\fake_system\reports\header1.html"

# data_file: The location and filename of the .csv file used to import the data.
data_file: "\etc\fake_system\reports\data_files\data1.csv"

# footer_file: The location and filename of the file the system must use to populate the report footer.
footer_file: "\etc\fake_system\reports\footer1.html"
```

## Optional Fields

For a complete list of optional configuration fields and their usage, see [Customizing Fake System Reports](https://google.com).

## Troubleshooting

The following error message is displayed if any of these fields are excluded, left blank, or include an incorrect path:

```
Error 240: Bad Config File.
```

Ensure that all fields are complete and point to the correct directories and files.

## Required Report Files

For more information about the specific files that the configuration file requires, see:

-   [Header Files](https://google.com)

-   [Data Files](https://google.com)

-   [Footer Files](https://google.com)
