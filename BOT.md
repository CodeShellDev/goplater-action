# Goplater Bot

Automate generated file updates in your repositories using [Goplater](https://github.com/CodeShellDev/goplater).

Goplater Bot is a reusable GitHub Actions workflow that runs Goplater, detects generated changes, and automatically commits and pushes them or creates a pull request.

> [!IMPORTANT]
> Looking for the **Goplater Action** instead?
> See [`goplater-action`](https://github.com/CodeShellDev/goplater-action)!

## Usage

The simplest setup is:

```yaml
name: Goplater Bot

on:
  push:
    branches:
      - main

jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
```

By default, the bot templates the current directory, commits and pushes generated changes directly, and creates a pull request if the direct push fails.

### Configure Goplater

You can pass the same configuration options as the Goplater Action:

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      path: templates/
      output: generated/
      recursive: true
      version: latest
```

### Create pull requests instead of pushing

If you prefer to review generated changes before they are applied, disable direct commits:

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      commit-and-push: false
      create-pr: true
```

The bot creates a pull request containing the generated changes.

## Inputs

### Goplater

See [`goplater-action`](https://github.com/CodeShellDev/goplater-action)!

### Git behavior

| Input             | Description                                                    | Default                      |
| ----------------- | -------------------------------------------------------------- | ---------------------------- |
| `commit-and-push` | Try to commit generated changes and push them directly         | `true`                       |
| `create-pr`       | Create a pull request if direct commit and push fails          | `true`                       |
| `branch`          | Branch to use when creating a pull request                     | `goplater/generated-updates` |
| `branch-suffix`   | Suffix appended to branch name when a new branch is made       | `""`                         |
| `base`            | Base branch for a pull request; defaults to the current branch | `""`                         |
| `dry-run`         | Generate changes without committing, pushing, or creating a PR | `false`                      |

### Commit identity

| Input               | Description                                                     | Default                         |
| ------------------- | --------------------------------------------------------------- | ------------------------------- |
| `commit-message`    | Commit message template                                         | `chore: update generated files` |
| `author-name`       | Git author name; defaults to the latest commit author           | `""`                            |
| `author-email`      | Git author email; defaults to the latest commit author          | `""`                            |
| `coauthor-username` | GitHub username or app slug used to resolve the co-author email | `goplater[bot]`                 |
| `coauthor-name`     | Co-author display name                                          | `goplater`                      |
| `coauthor-email`    | Co-author email; overrides the resolved noreply email           | `""`                            |

### Pull requests

| Input                | Description                                                       | Default                         |
| -------------------- | ----------------------------------------------------------------- | ------------------------------- |
| `update-existing-pr` | Update an existing open Goplater PR instead of creating a new one | `false`                         |
| `pr-title`           | Pull request title template                                       | `chore: update generated files` |
| `pr-body`            | Pull request body template                                        | See default template            |
| `pr-assignees`       | Comma-separated GitHub usernames to assign                        | `""`                            |
| `pr-reviewers`       | Comma-separated GitHub usernames to request reviews from          | `""`                            |
| `pr-labels`          | Comma-separated labels                                            | `""`                            |
| `pr-draft`           | Create the pull request as a draft                                | `false`                         |
| `delete-branch`      | Delete the generated branch after the PR is merged                | `true`                          |

### GitHub App

| Input       | Description                                         | Default |
| ----------- | --------------------------------------------------- | ------- |
| `client-id` | GitHub App client ID used to create an access token | `""`    |

## Pull request behavior

Goplater Bot supports two different pull request strategies.

### Create a new pull request each time

By default, every workflow run gets its own generated branch and can create a separate pull request.

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      commit-and-push: false
      create-pr: true
      update-existing-pr: false
```

For example, the generated branches look like:

```
goplater/generated-updates-12345
goplater/generated-updates-12346
goplater/generated-updates-12347
```

This is useful when you want to review each generation independently.

### Update an existing pull request

Enable `update-existing-pr` to reuse the same generated branch and update its existing open pull request.

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      commit-and-push: false
      create-pr: true
      update-existing-pr: true
```

All subsequent workflow runs build on top of the same branch:

```
goplater/generated-updates
```

This is useful for repositories where generated changes are frequent and you prefer maintaining a single pull request.

The existing pull request is updated with the latest generated changes instead of creating another one.

## Commit and push

By default, Goplater Bot attempts to commit and push generated changes directly to the triggering branch.

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      commit-and-push: true
```

If the push fails and `create-pr` is enabled, the bot falls back to creating a pull request.

You can also disable both options if you only want to generate and inspect the changes.

## Commit identity

Goplater Bot allows you to customize the author and co-author of generated commits.

By default, the author is taken from the latest commit in the repository.

The co-author defaults to `goplater[bot]`.

You can override these values:

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      author-name: My Name
      author-email: my-email@example.com
      coauthor-username: goplater[bot]
      coauthor-name: Goplater
```

The co-author email is automatically resolved using the GitHub account's noreply email unless `coauthor-email` is explicitly provided.

## Template variables

The commit message, PR title, and PR body support template variables.

| Variable                    | Description                                       |
| --------------------------- | ------------------------------------------------- |
| `{{ repository }}`          | Repository name (`owner/repository`)              |
| `{{ repository_url }}`      | Repository URL                                    |
| `{{ server_url }}`          | GitHub server URL                                 |
| `{{ base }}`                | Base branch                                       |
| `{{ branch }}`              | Generated branch                                  |
| `{{ branch_url }}`          | Generated branch URL                              |
| `{{ actor }}`               | User who triggered the workflow                   |
| `{{ actor_url }}`           | User profile URL                                  |
| `{{ sha }}`                 | Commit SHA                                        |
| `{{ short_sha }}`           | Short commit SHA                                  |
| `{{ sha_url }}`             | Commit URL                                        |
| `{{ run_id }}`              | Workflow run ID                                   |
| `{{ run_url }}`             | Workflow run URL                                  |
| `{{ workflow }}`            | Workflow name                                     |
| `{{ event }}`               | Triggering event                                  |
| `{{ goplater_version }}`    | Actual Goplater release tag used (e.g. `v1.2.3`)  |
| `{{ date }}`                | Current UTC date (`YYYY-MM-DD`)                   |
| `{{ changed_files_count }}` | Total number of changed files                     |
| `{{ added_count }}`         | Number of added files                             |
| `{{ modified_count }}`      | Number of modified files                          |
| `{{ deleted_count }}`       | Number of deleted files                           |
| `{{ changed_files }}`       | Comma-separated list of changed files             |
| `{{ changed_files_list }}`  | Bullet list of changed files, linked in PR bodies |

### Customize the commit message

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      commit-message: "chore: update {{ changed_files_count }} generated files"
```

### Customize the pull request

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      pr-title: "chore: update generated files ({{ changed_files_count }})"
      pr-body: |
        # Generated Updates

        Goplater generated {{ changed_files_count }} file change(s).

        ## Changed files

        {{ changed_files_list }}

        [View workflow run]({{ run_url }})
```

## GitHub App authentication

By default, the workflow uses the automatically provided `GITHUB_TOKEN`.

You can optionally use a GitHub App to authenticate the workflow.

This is useful when you want generated commits and pull requests to trigger other GitHub Actions workflows.

Create a GitHub App with the following repository permissions:

- Contents: Read and write
- Pull requests: Read and write

Then configure the workflow:

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      client-id: ${{ vars.GOPLATER_APP_CLIENT_ID }}
    secrets:
      app_private_key: ${{ secrets.GOPLATER_APP_PRIVATE_KEY }}
```

Alternatively, you can provide a personal access token or GitHub App token:

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    secrets:
      token: ${{ secrets.GH_TOKEN }}
```

When both a GitHub App and a token are configured, the GitHub App token takes precedence.

## Secrets

| Secret            | Description                                           |
| ----------------- | ----------------------------------------------------- |
| `app_private_key` | GitHub App private key used to create an access token |
| `token`           | Personal access token or GitHub App token             |

## Outputs

The reusable workflow exposes the following outputs:

| Output                | Description                                   |
| --------------------- | --------------------------------------------- |
| `changed`             | Whether Goplater generated changes            |
| `pushed`              | Whether changes were pushed directly          |
| `pull-request-url`    | URL of the created or updated pull request    |
| `pull-request-number` | Number of the created or updated pull request |

You can use these outputs in subsequent jobs:

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1

  report:
    needs: goplater
    runs-on: ubuntu-latest
    if: needs.goplater.outputs.changed == 'true'
    steps:
      - run: echo "Generated changes detected!"
```

## Dry run

Use `dry-run` to generate changes without committing, pushing, or creating a pull request.

```yaml
jobs:
  goplater:
    uses: CodeShellDev/goplater-action/.github/workflows/goplater-bot.yml@v1
    with:
      dry-run: true
```

The generated changes, commit message, PR title, and PR body are displayed in the workflow logs and summary.

## Loop prevention

Goplater Bot automatically adds the following trailer to generated commits:

```text
Goplater-Generated: true
```

The workflow checks for this marker and ignores its own generated commits.

It also ignores events originating from the configured generated PR branch.

This prevents the bot from repeatedly triggering itself.

## Requirements

The workflow must be called from a repository that allows reusable workflows and provides the required permissions.

The calling workflow should grant:

```yaml
permissions:
  contents: write
  pull-requests: write
```

Goplater Bot checks out the repository automatically, downloads the requested Goplater release, and handles generated changes without requiring additional setup.
