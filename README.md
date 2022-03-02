# Github Action to start a gitflow release

This action starts a gitflow release, and opens a pull request for the release.

# Usage

## Example

Here is an example of what to put in your `.github/workflows/start-release.yml` file to use this workflow.

```yaml
name: Start Release
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Next version number'
        required: true
env:
  git_user: some-bot                # CHANGE ME!
  git_email: some-bot@example.com   # CHANGE ME!
jobs:
  start-release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: develop                # Always release from develop
          fetch-depth: 0
          persist-credentials: false  # Needed so we can push with different credentials.
                                      # NOTE: Pushing with different credentials allows admins to push protected branches.
                                      # NOTE: Pushing with different credentials allow workflows to trigger from the push.

      # START THE RELEASE
      - name: Initialize mandatory git config
        run: |
          git config --global user.name $git_user &&
          git config --global user.email $git_email
      - name: Start gitflow release
        uses: duplocloud/ghactions-start-gitflow-release@master
        with:
          github_token: ${{ secrets.RELEASE_BOT_GITHUB_TOKEN }}
          version: ${{ github.event.inputs.version }}
          precommit_run: |
            echo "v${{ github.event.inputs.version }}" >VERSION
```

## Inputs

Unlike standard actions, this action just uses variables from the environment.

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| version | The release version (without any leading prefix) | true | unset |
| tag_prefix | The tag prefix (defaults to "v") | true | `"v"` |
| github_token | Github token to use for pushing and creating the PR | true | unset |
| precommit_run | Shell commands to run prior to making release commit.  If no commands are given, no commit will be made. | false | unset |
| commit_message | Commit message to use for the release commit.  If no precommit_run is given, no commit will be made. | false | `"[release] version bump"` |
| create_pr | Whether or not to create a pull request. | false | true |
| target_ref | pull request base (defaults to master).  Only used if creating a PR. | false | `"master"` |

## Outputs

| Name | Description |
|------|-------------|
| release_tag | release git tag (with version prefix) |
| release_branch | release git branch |
| pull_request_id | pull request ID |
| pull_request_url | pull request URL |
