# github-workflows-release-drafter

This workflow will create a release draft based on the PRs merged since the last release. It will also create a changelog based on the PRs merged.

This workflow will also try to assign labels to PR based on the title of the PR. The labels is defined [here](.github/release-drafter.yml)

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
      coopnorge/github-workflows-release-drafter/.github/workflows/release-drafter.yaml@9d47c40559c78b2e314357fd99a5eca428eb5481
    secrets: inherit
```

2. Create a `.github/release-drafter.yml` file in your repository.
Copy the [default configuration](.github/release-drafter.yml) and modify it to your needs.

## How it works

When a PR is created, this workflow will check the PR title and try to assign labels to the PR based on the title. If you do not agree with the labels assigned you can assign new labels yourself.

Supported labels for version bumping (only one label is taken into account, in order):

```
- "major" # bump X.0.0
- "minor" # bump 0.X.0
- "patch" # bump 0.0.X
```

Supported labels to help with changelog grouping:
  
```
- "dependency"
- "chore"
- "enhancements"
- "bug-fix"
- "deprecation"
- "breaking-change"
```

After you have merged the PR, the draft release will be created/updated. The draft release will be based on the PRs merged since the last release. The draft release will also contain a changelog based on the PRs merged.

When you are ready to release, you can publish the release draft. This will create a new release and tag the release with the version number.

**NOTE:** For golang project(`release-drafter-go.yaml`), the workflow try to check if exported interfaces are compatible or not using, [apidiff](https://pkg.go.dev/golang.org/x/exp/cmd/apidiff). If any exported interfaces is not compatible, the workflow will add a label `major` to the PR. If all exported interfaces are compatible, the workflow will add a label `minor` to the PR. And if there are no changes to exported interfaces, the workflow will add a label `patch` to the PR.

## Configuration

### release-drafter.yml

Supported configuration options:

| Option | Description| Required | Default |
| :---: | :--- | :---: | :---: |
| `config-name` | The name of the configuration file to use| `false` | `release-drafter.yml` |
| `commitish` | Commitish to use when calculating the version bump. Defaults to the pull request base branch. | `false` | `main` |
| `publish` | Publish the release draft when the PR is merged. | `false` | `false` |

### release-drafter-go.yml

Supported configuration options:

| Option | Description| Required | Default |
| :---: | :--- | :---: | :---: |
| `config-name` | The name of the configuration file to use| `false` | `release-drafter.yml` |
| `commitish` | Commitish to use when calculating the version bump. Defaults to the pull request base branch. | `false` | `main` |
| `publish` | Publish the release draft when the PR is merged. | `false` | `false` |
| `project-path` | The path to go project. | `false` | `.` |