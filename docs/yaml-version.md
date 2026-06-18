# YAML Version

The **YAML Version Check** workflow `yaml-version.yml` checks that the YAML version used in the project is the same as the one used for releases using the `version.yml` workflow.
Typically for Dart/Flutter projects.

## Pre-requisites

1. The workflow expects a `.yaml` file in the provided `yaml_file` variable (including path to) to check the YAML version
2. The workflow expects a the `tag` output of the `version.yml` workflow.

## Operation

You need to run the `version.yml` workflow before this one, and provide the `tag` output as input.

The workflow will check the YAML version against the `tag` and:

- succeed if the YAML version is the same as the `tag`
- if the YAML version is different from the `tag`:
  - fail if it's a release build
  - succeed with a warning if it's a development build

Example:

```yaml
jobs:
  version:
    uses: nxlabs-ch/shared-workflows/.github/workflows/version.yml@main

  check-yaml-version:
    needs: version
    uses: nxlabs-ch/shared-workflows/.github/workflows/yaml-version.yml@main
    with: 
      version: ${{ needs.version.outputs.tag }}
      yaml_file: my_module/pubspec.yaml
```
