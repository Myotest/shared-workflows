# JSON Version

The **JSON Version Check** workflow `json-version.yml` checks that the JSON version used in the project is the same as the one used for releases using the `version.yml` workflow.
Typically for Dart/Flutter projects.

## Pre-requisites

1. The workflow expects a `.json` file in the provided `json_file` variable (including path to) to check the JSON version
2. The workflow expects a the `tag` output of the `version.yml` workflow.

## Operation

You need to run the `version.yml` workflow before this one, and provide the `tag` output as input.

The workflow will check the JSON version against the `tag` and:

- succeed if the JSON version is the same as the `tag`
- if the JSON version is different from the `tag`:
  - fail if it's a release build
  - succeed with a warning if it's a development build

Example:

```json
jobs:
  version:
    uses: nxlabs-ch/shared-workflows/.github/workflows/version.yml@main

  check-json-version:
    needs: version
    uses: nxlabs-ch/shared-workflows/.github/workflows/json-version.yml@main
    with: 
      version: ${{ needs.version.outputs.tag }}
      json_file: my_module/plugin.json
```
