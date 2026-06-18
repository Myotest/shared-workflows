# Check Version Action

The **Check Version** composite action `.github/actions/check-version` contains the shared implementation used by all version-check workflows. It can also be used directly in any workflow step when you need finer control over the job structure.

## Inputs

| Input         | Required | Default  | Description                                                                  |
| ------------- | -------- | -------- | ---------------------------------------------------------------------------- |
| `version`     | yes      |          | The TAG version string for this release (e.g. `v1.2.3`)                      |
| `file`        | yes      |          | Path to the file from which to extract the version                           |
| `extract_cmd` | yes      |          | Shell command to extract the version string; `$FILE` is set to the file path |
| `label`       | no       | `"File"` | Human-readable label shown in log output (e.g. `Rust`, `TOML`)               |

## Operation

The action runs four steps:

1. Strips the `v` prefix from `version` to obtain the expected version string
2. Runs `extract_cmd` in a shell with `$FILE` set to `file`, capturing stdout as the extracted version
3. Prints both versions to the log
4. Compares them:
   - Passes if they match
   - Fails on a mismatch for a release build (non-empty expected version)
   - Passes with a warning on a mismatch for a development build (empty expected version)

Note: the action does not perform a `checkout`. The calling workflow or job must check out the repository before invoking it.

## Using the action directly

Call the action from a step with `uses: ./.github/actions/check-version`. The action is pinned to `@main` so it always tracks the latest version without requiring a versioned release of the action itself.

```yaml
jobs:
  version:
    uses: nxlabs-ch/shared-workflows/.github/workflows/version.yml@main

  check-my-version:
    runs-on: ubuntu-24.04
    needs: version
    steps:
      - uses: actions/checkout@v6
      - uses: nxlabs-ch/shared-workflows/.github/actions/check-version@main
        with:
          version: ${{ needs.version.outputs.tag }}
          file: src/config.toml
          extract_cmd: "head -n 10 \"$FILE\" | grep version -m 1 | sed 's/\"/ /g' | awk '{print $3}'"
          label: Config
```

This is equivalent to using `check-version.yml` as a reusable workflow but keeps the version check as a step inside your own job, which can be useful when you need to run it alongside other steps.
