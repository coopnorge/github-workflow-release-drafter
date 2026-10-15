# Release Drafter GitHub Workflow

This workflow will create a release draft based on the PRs merged since the last
release. It will also create a changelog based on the PRs merged.

It will also try to assign labels to PR based on the title of the PR. The labels
are defined in the
[default configuration](https://github.com/coopnorge/github-workflow-release-drafter/blob/main/.github/release-drafter.yml)

## Usage

1. Create a `.github/workflows/release-drafter.yaml` file in your repository

```yaml
name: Release Drafter
on:
  push:
    branches:
      - main
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
      - edited
permissions:
  contents: read
jobs:
  release-draft:
    permissions:
      pull-requests: write
      contents: write
      issues: write
    uses: >-
      coopnorge/github-workflow-release-drafter/.github/workflows/release-drafter.yaml@v0.8.0
```

**OR** for Go projects

```yaml
name: Release Drafter
on:
  push:
    branches:
      - main
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
      - edited
permissions:
  contents: read
jobs:
  release-draft:
    permissions:
      pull-requests: write
      contents: write
      issues: write
    uses: coopnorge/github-workflow-release-drafter/.github/workflows/release-drafter-go.yaml@v0.8.0
    with:
      project-path: "./go-playground"
    secrets: inherit
```

> [!NOTE]
>
> - As we generally have projects inside subdirectories, we need to specify the
>   `project-path`.
> - Make sure to request permissions.
> - `secrets: inherit` should be used if you reference other internal/private
>   repositories.

2. Optionally, customize the configuration. If you do nothing, the default
   configuration is used (see below).

If neither your repository nor your organisation's `.github` repository has the
config file (`config-name`), the workflow falls back to the
[default configuration](https://github.com/coopnorge/github-workflow-release-drafter/blob/main/.github/release-drafter.yml)
from this repository, at the same version as the workflow you pinned in `uses:`.

### Customizing the default configuration

Do not copy the default configuration. Instead, create a
`.github/release-drafter.yml` file that extends it. Keys you set replace the
inherited ones:

```yaml
_extends: coopnorge/github-workflow-release-drafter:/.github/release-drafter.yml@v0.8.0
```

To add entries to list keys such as `categories` or `autolabeler` without
repeating the inherited ones, use the mapping form with the `append` (or
`prepend`) strategy:

```yaml
_extends:
  from: coopnorge/github-workflow-release-drafter:/.github/release-drafter.yml@v0.8.0
  strategy:
    categories: append
categories:
  - type: "pre-exclude"
    when:
      label: "skip-changelog"
```

> [!NOTE]
>
> The `@ref` in `_extends` is independent of the version in your `uses:` line.
> Keep them in sync, or omit `@ref` to follow the default branch.

See the
[release-drafter configuration loading docs](https://github.com/release-drafter/release-drafter/blob/v7.7.0/docs/configuration-loading.md)
for details.

## How it works

When a PR is created, this workflow will check the PR title and try to assign
labels to the PR based on the title. If you do not agree with the labels
assigned you can assign new labels yourself.

Supported labels for version bumping (only one label is taken into account, in
order):

- `major` (bump X.0.0)
- `minor` (bump 0.X.0)
- `patch` (bump 0.0.X)

Supported labels to help with changelog grouping:

- `dependency`
- `chore`
- `enhancements`
- `bug-fix`
- `deprecation`
- `breaking-change`

After you have merged the PR, the draft release will be created/updated. The
draft release will be based on the PRs merged since the last release. The draft
release will also contain a changelog based on the PRs merged.

When you are ready to release, you can publish the release draft. This will
create a new release and tag the release with the version number.

> [!NOTE]
>
> For Go projects (`release-drafter-go.yaml`), the workflow try to check if
> exported interfaces are compatible or not using,
> [apidiff](https://pkg.go.dev/golang.org/x/exp/cmd/apidiff). If any exported
> interfaces is not compatible, the workflow will add a label `major` to the PR.
> If all exported interfaces are compatible, the workflow will add a label
> `minor` to the PR. And if there are no changes to exported interfaces, the
> workflow will add a label `patch` to the PR.

## Configuration

### release-drafter.yml

Required Secret:

|          Secret          | Description                                                                                                                                                                          |
| :----------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `REVIEWBOT_GITHUB_TOKEN` | A GitHub personal access token with `repo` scope. This secret should be defined in the caller repo or at the org level. It must be inherited in the workflow with `secrets: inherit` |

Supported configuration options:

|    Option     | Description                                                                                | Required |        Default        |
| :-----------: | :----------------------------------------------------------------------------------------- | :------: | :-------------------: |
| `config-name` | The name of the configuration file to use                                                  | `false`  | `release-drafter.yml` |
|  `commitish`  | Commit to use when calculating the version bump. Defaults to the pull request base branch. | `false`  |        `main`         |
|   `publish`   | Publish the release draft when the PR is merged.                                           | `false`  |        `false`        |

### release-drafter-go.yml

Required Secret:

|          Secret          | Description                                                                                                                                                                          |
| :----------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `REVIEWBOT_GITHUB_TOKEN` | A GitHub personal access token with `repo` scope. This secret should be defined in the caller repo or at the org level. It must be inherited in the workflow with `secrets: inherit` |

Supported configuration options:

|               Option               | Description                                                                                | Required |        Default        |
| :--------------------------------: | :----------------------------------------------------------------------------------------- | :------: | :-------------------: |
|           `config-name`            | The name of the configuration file to use                                                  | `false`  | `release-drafter.yml` |
|            `commitish`             | Commit to use when calculating the version bump. Defaults to the pull request base branch. | `false`  |        `main`         |
|             `publish`              | Publish the release draft when the PR is merged.                                           | `false`  |        `false`        |
|           `project-path`           | The path to go project.                                                                    | `false`  |          `.`          |
| `ignore-major-changes-for-pattern` | A regular expression pattern to match against incompatible changes.                        | `false`  |          `.`          |
