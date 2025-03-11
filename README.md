# Reusable Github Action Workflows

## Workflows

### Auto merge dependabot

This workflow will approve and merge dependabot pull requests, assuming you've setup additional checks that will test if the dependency works.

#### Requirements:

This action requires a PAT with the scope `repo` for private repositories or `public_repo` for public repositories.

> [!NOTE]
> The token *MUST* be created from a user with push permission to the repository.

#### Usage:

The following workflow will execute the action each workday at 7:30.

```yaml
name: Auto merge dependabot pull requests

on:
  schedule:
    - cron: "30 7 * * 1-5"

jobs:
  auto-merge-dependabot:
    uses: manuelvanrijn/reusable_github_action_workflows/.github/workflows/auto-merge-dependabot.yml@main
    secrets:
      personal_access_token: ${{ secrets.PAT_TOKEN }}
```
