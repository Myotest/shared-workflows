# Version

The **Semantic Version Management** workflow `version.yml` allows handling automatic version changes according to [semantic versioning](https://semver.org).

## Pre-requisites

1. The release branch should be named `main`
2. The default branch should be named `develop`
3. There must be at least one TAG in the repo, for example `v0.0.0`
4. You should protect the `main` branch, at minimum with the following rules:
   1. "Require a pull request before merging" and "Require approvals"
   2. "Restrict who can push to matching branches"
5. You will need a personal access token (PAT) with `repo` scope, stored as a secret named `FF_MERGE_PAT` in your
   repository settings, in order to be able to fast-forward merge `main` back into `develop` after a release.

Items 1. and 2. are a subset of [GitFlow](https://nvie.com/posts/a-successful-git-branching-model/).

## Operation

Inputs:

- `preface`: The file path whose content to use as preface for the release notes, in markdown format.
- `postface`: The file path whose content to use as postface for the release notes, in markdown format.
- `tag_prefix`: A prefix to the tag name (default: `v`).

You should run `version` as the first job in your workflow.
It will do the following:

1. If it is a PR to `main`:
   1. Compute the future version based on commit messages. (see **Commit Message Format** below).
   2. Add a comment to the PR with the future version, tag and change log.
2. If it is a push to `main`:
   1. TAG the branch with the new release tag.
   2. Create a GitHub release with the changelog
   3. Merge `main` back into `develop` with a fast-forward merge.
3. Provide the current version number and tag as an output:
   1. This is the result of `git describe --tags --dirty`.
   2. For a normal build this will be something like:
      1. version: `1.0.2-4-g66f9fca`
      2. tag: `v1.0.2-4-g66f9fca`
      3. pep440: `1.0.2.dev4`
   3. For a release build this will be the new version/tag, something like:
      1. version: `1.0.3`
      2. tag: `v1.0.3`
      3. pep440: `1.0.3`

## Required Permissions

In your repository, go to **Settings → Actions → General → Workflow permissions** and select **"Read and write permissions"**.

Without this, the workflow will fail with a `403 Resource not accessible by integration` error when trying to post a PR comment.

> **Important:** The settings page has separate **Save** buttons for each section. Make sure to click the Save button in the **Workflow permissions** section specifically.
> **Note:** For organization repositories, this is typically set at the organization level and does not need to be configured per-repository.

## Example

``` yaml
jobs:
  version:
    uses: nxlabs-ch/shared-workflows/.github/workflows/version.yml@main
    with:
      ...
    secrets:
      FF_MERGE_PAT: ${{ secrets.FF_MERGE_PAT }}
```

## Commit Message Format

To support automatic versioning, commit messages should be in the following format:

```txt
<type>: <short summary>

<body>
```

With the following `<type>`:
  
| Type       | Change | Changelog Section        | Description                                                      |
|------------|--------|--------------------------|------------------------------------------------------------------|
| api        | minor  | API Changes              | Backward compatible API change                                   |
| breaking   | major  | -                        | Incompatible API change or other<br>breaking change              |
| build      | patch  | Build Systems            | Changes that affect the build system<br>or external dependencies |
| ci         | patch  | Continuous Integration   | Changes to the CI configuration files<br>and scripts             |
| doc / docs | patch  | Documentation            | Documentation only changes                                       |
| feature    | minor  | Features                 | A new feature                                                    |
| fix        | patch  | Bug Fixes                | A bug fix                                                        |
| perf       | patch  | Performance Improvements | A code change that improves performance                          |  
| refactor   | patch  | Code Refactoring         | A code change that neither fixes a bug<br>nor adds a feature     |  
| revert     | minor  | Reverts                  | Revert of a previous change                                      |
| test       | patch  | Tests                    | Adding missing tests or correcting existing tests                |

For each `type` the corresponding semantic version `change` would be incremented. The biggest wins.

## Attribution

The workflow uses the [amplifysa/github-tag-action](https://github.com/amplifysa/github-tag-action) action.
