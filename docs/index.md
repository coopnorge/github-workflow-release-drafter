# Release Drafter GitHub Workflow

This workflow will create a release draft based on the PRs merged since
the last release. It will also create a changelog based on the PRs merged.

It will also try to assign labels to PR based on the title of the PR.
The labels is defined [here](https://github.com/coopnorge/github-workflow-release-drafter/blob/main/.github/release-drafter.yml)

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
    uses: >-
      coopnorge/github-workflow-release-drafter/.github/workflows/release-drafter.yaml@v0.1.0
```

**OR** for golang project

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
    uses: coopnorge/github-workflow-release-drafter/.github/workflows/release-drafter-go.yaml@v0.1.0
    with:
      project-path: "./go-playground"
    secrets:
      GITHUB_OAUTH_TOKEN: ${{ secrets.OUR_GITHUB_TOKEN }}
```

**NOTE:** As we generally have projects inside subdirectories, we need to
specify the `project-path`.

**NOTE:** Make sure to request permissions

**NOTE:** GITHUB_OAUTH_TOKEN should be supplied if you reference other 
internal/private repositories


```yaml
pull-requests: write
contents: write
```

2. Create a `.github/release-drafter.yml` file in your repository.
Copy the
[default configuration](https://github.com/coopnorge/github-workflow-release-drafter/blob/main/.github/release-drafter.yml)
and modify it to your needs.

## How it works

When a PR is created, this workflow will check the PR title and
try to assign labels to the PR based on the title. If you do not agree
with the labels assigned you can assign new labels yourself.

Supported labels for version bumping (only one label is taken into account, in order):

- "major" (bump X.0.0)
- "minor" (bump 0.X.0)
- "patch" (bump 0.0.X)

Supported labels to help with changelog grouping:
  
- "dependency"
- "chore"
- "enhancements"
- "bug-fix"
- "deprecation"
- "breaking-change"

After you have merged the PR, the draft release will be created/updated.
The draft release will be based on the PRs merged since the last release.
The draft release will also contain a changelog based on the PRs merged.

When you are ready to release, you can publish the release draft.
This will create a new release and tag the release with the version number.

**NOTE:** For golang project(`release-drafter-go.yaml`),
the workflow try to check if exported interfaces are compatible or not using,
[apidiff](https://pkg.go.dev/golang.org/x/exp/cmd/apidiff).
If any exported interfaces is not compatible,
the workflow will add a label `major` to the PR.
If all exported interfaces are compatible,
the workflow will add a label `minor` to the PR.
And if there are no changes to exported interfaces,
the workflow will add a label `patch` to the PR.

## Configuration

### release-drafter.yml

Required Secret:

| Secret | Description |
| :---: | :--- |
| `PERSONAL_ACCESS_TOKEN` | A GitHub personal access token with `repo` scope. (Pass: `secrets.REVIEWBOT_GITHUB_TOKEN`) |

Supported configuration options:

| Option | Description| Required | Default |
| :---: | :--- | :---: | :---: |
| `config-name` | The name of the configuration file to use| `false` | `release-drafter.yml` |
| `commitish` | Commitish to use when calculating the version bump. Defaults to the pull request base branch. | `false` | `main` |
| `publish` | Publish the release draft when the PR is merged. | `false` | `false` |

### release-drafter-go.yml

Required Secret:

| Secret | Description |
| :---: | :--- |
| `PERSONAL_ACCESS_TOKEN` | A GitHub personal access token with `repo` scope. (Pass: `secrets.REVIEWBOT_GITHUB_TOKEN`) |

Supported configuration options:

| Option | Description| Required | Default |
| :---: | :--- | :---: | :---: |
| `config-name` | The name of the configuration file to use| `false` | `release-drafter.yml` |
| `commitish` | Commitish to use when calculating the version bump. Defaults to the pull request base branch. | `false` | `main` |
| `publish` | Publish the release draft when the PR is merged. | `false` | `false` |
| `project-path` | The path to go project. | `false` | `.` |
| `ignore-major-changes-for-pattern` | A regular expression pattern to match against incompatible changes. | `false` | `.` |
