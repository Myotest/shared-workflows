# Check Version

The **Check Version** workflow `check-version.yml` is the consolidated entry point for verifying that a version string in any source file matches a release TAG. All format-specific version workflows (`c-version`, `rust-version`, etc.) are thin wrappers around this workflow.

The shared logic lives in a composite action at `.github/actions/check-version/action.yml`. Both `check-version.yml` and the format-specific wrappers delegate to it via `uses: ./.github/actions/check-version`, which GitHub resolves against the shared-workflows repository at the same ref as the calling workflow — no version pin required in the wrappers.

## Inputs

| Input         | Required | Default  | Description                                                           |
| ------------- | -------- | -------- | --------------------------------------------------------------------- |
| `version`     | yes      |          | The TAG version string for this release (e.g. `v1.2.3`)               |
| `file`        | yes      |          | Path to the file from which to extract the version                    |
| `extract_cmd` | yes      |          | Shell command to extract the version string; `$FILE` is set to `file` |
| `label`       | no       | `"File"` | Human-readable label shown in log output (e.g. `Rust`, `TOML`)        |

## Operation

The workflow:

1. Checks out the repository
2. Strips the `v` prefix from `version` to get the expected version string
3. Runs `extract_cmd` with `$FILE` set to the `file` input and captures its stdout as the extracted version
4. Compares the extracted version against the expected version:
   - Passes if they match
   - Fails if they differ on a release build (non-empty expected version)
   - Passes with a warning if the expected version is empty (pre-release / development build)

## Writing an `extract_cmd`

The command receives the file path via `$FILE` and must print the bare version string (e.g. `1.2.3`) to stdout. Any standard shell pipeline is valid.

Common patterns used by the format-specific wrappers:

| Format | `extract_cmd`         |                     |                       |                   |                   |
| ------ | --------------------- | ------------------- | --------------------- | ----------------- | ----------------- |
| TOML   | `head -n 10 "$FILE" \ | grep version -m 1 \ | sed 's/"/ /g' \       | awk '{print $3}'` |                   |
| JSON   | `head -n 10 "$FILE" \ | grep version -m 1 \ | sed 's/"/ /g' \       | awk '{print $3}'` |                   |
| YAML   | `head -n 10 "$FILE" \ | grep version -m 1 \ | awk '{print $2}'`     |                   |                   |
| Rust   | `head -n 6 "$FILE" \  | grep version -m 1 \ | sed 's/"/ /g' \       | awk '{print $3}'` |                   |
| SPEC   | `tail -n 10 "$FILE" \ | grep version -m 1 \ | sed "s/'/ /g" \       | sed 's/"/ /g' \   | awk '{print $2}'` |
| KiCad  | `head -n 15 "$FILE" \ | grep rev \          | sed 's/"/ /g' \       | awk '{print $2}'` |                   |
| C      | `head -n 10 "$FILE" \ | grep define \       | grep 'VERSION' -m 1 \ | sed 's/"/ /g' \   | awk '{print $3}'` |

## Example

Direct use with a custom extraction command:

```yaml
jobs:
  version:
    uses: nxlabs-ch/shared-workflows/.github/workflows/version.yml@main

  check-my-version:
    needs: version
    uses: nxlabs-ch/shared-workflows/.github/workflows/check-version.yml@main
    with:
      version: ${{ needs.version.outputs.tag }}
      file: src/config.toml
      extract_cmd: "head -n 10 \"$FILE\" | grep version -m 1 | sed 's/\"/ /g' | awk '{print $3}'"
      label: Config
```

## Format-specific wrappers

For common file formats, convenience wrappers are available that pre-fill `extract_cmd` and `label`. Prefer these when the format matches; use `check-version.yml` directly for custom or uncommon formats.

| Workflow            | Key input                       |
| ------------------- | ------------------------------- |
| `c-version.yml`     | `header_file`, `version_define` |
| `json-version.yml`  | `json_file`                     |
| `kicad-version.yml` | `kicad_sch_path`                |
| `rust-version.yml`  | `cargo_toml_dir`                |
| `spec-version.yml`  | `spec_file`                     |
| `toml-version.yml`  | `toml_file`                     |
| `yaml-version.yml`  | `yaml_file`                     |
