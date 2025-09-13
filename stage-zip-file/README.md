# Stage & Zip WP Plugin

A GitHub Composite Action that **stages WordPress plugin files** into a `.release` folder and creates a **versioned zip file** ready for deployment or distribution.

This action is designed to standardize the packaging process across multiple WordPress plugins, ensuring consistent builds with minimal workflow code.

---

## Features

- Stages plugin source files into a clean `.release` directory.
- Supports **multiline array-style inputs** for directories and extra files.
- Case-insensitive detection of `README.txt` (optional).
- Always includes the main plugin file (required).
- Generates a versioned zip (`<slug>-<tag>.zip`) for release.
- Outputs the full path of the generated zip for easy use in later steps.

---

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `plugin_file` | ✅ | — | Main plugin file (e.g., `myplugin.php`). |
| `plugin_version` | ✅ | — | Normalized version tag (e.g., `1.2.3`). |
| `plugin_slug` | ✅ | — | Plugin slug used for naming the zip file. |
| `stage_dirs` | ❌ | *(empty)* | Multiline list of directories to stage if they exist. |
| `extra_files` | ❌ | *(empty)* | Multiline list of extra files to stage if they exist. |
| `readme_case_insensitive` | ❌ | `true` | Whether to detect `README.txt` in a case-insensitive way. |
| `release_dir` | ❌ | `.release` | Temporary staging directory. |
| `zip_outdir` | ❌ | `.` | Directory where the zip file will be created. |

---

## Outputs

| Name | Description |
| --- | --- |
| `zip_path` | Full path to the generated plugin zip file. |

---

## Example Usage

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate version
        id: validate
        uses: airygen/action-wordpress-plugin-release-check@v1
        with:
          plugin_file: myplugin.php

      - name: Stage & Zip
        id: pack
        uses: airygen/actions/stage-and-zip-wp-plugin@v1
        with:
          plugin_file: myplugin.php
          wp_slug: myplugin
          tag: ${{ steps.validate.outputs.tag }}
          stage_dirs: |
            languages
            inc
            src
            vendor
          extra_files: |
            uninstall.php
            LICENSE
            composer.json
          readme_case_insensitive: "true"

      - name: Attach zip to GitHub Release
        uses: softprops/action-gh-release@v2
        with:
          files: ${{ steps.pack.outputs.zip_path }}
```

---

## Notes

Use this action together with `validate-plugin-version` for a full release pipeline (validation → staging → deployment).

`stage_dirs` and `extra_files` use multiline strings to simulate arrays in YAML.

The generated `.release` folder is temporary and can be used as the `build-dir` for deploy actions