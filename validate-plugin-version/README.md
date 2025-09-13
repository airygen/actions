# Validate WP Plugin Version

A GitHub Composite Action that validates the consistency of version numbers when publishing a WordPress plugin.  
It ensures that:

- The **Release Tag** (e.g., `v1.2.3` or `1.2.3`)  
- The **Stable tag** in `README.txt`  
- The **Version header** in the main plugin file  

are consistent before deploying to WordPress.org.

---

## Features

- Strips a leading `v` from release tags automatically (`v1.2.3` → `1.2.3`).  
- Case-insensitive detection of `README.txt`.  
- Validates:
  - `README.txt` exists and contains a `Stable tag`.  
  - The plugin main file exists and contains a `Version:` header.  
  - `Stable tag` matches the release tag (or is set to `trunk`).  
  - Plugin header version matches the release tag.  
- Fails early with a clear error message if checks do not pass.  
- Exposes two outputs:
  - `tag` → normalized version number  
  - `publish` → `"true"` or `"false"`

---

## Inputs

| Name | Required | Description 
|--- | --- | --- |
| `plugin_file` | ✅ | The main plugin file (e.g., `airyseo.php`). |

---

## Outputs

| Name | Description|
| --- | --- |
| `tag` | Normalized version (release tag without the leading `v`). |
| `publish` | Whether the plugin can be deployed (`"true"` or `"false"` as a string). |

---

## Example Usage

```yaml
name: Build and Deploy

on:
  release:
    types: [published]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate WP Plugin Version
        id: validate
        uses: airygen/action-wordpress-plugin-release-check@v1
        with:
          plugin_file: airyseo.php

      - name: Use validation outputs
        if: steps.validate.outputs.publish == 'true'
        run: |
          echo "Version is valid!"
          echo "Normalized tag: ${{ steps.validate.outputs.tag }}"
```

---

## Notes

- This action is intended to be used in **release workflows** triggered by `on: release`.  
- Always check `steps.<id>.outputs.publish` before deploying to WordPress.org.  
- Outputs are **strings**, so compare them with `'true'` or `'false'` explicitly.  