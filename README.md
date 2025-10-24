# Reusable Github Action Workflows

## Workflows

- [Auto merge dependabot](#auto-merge-dependabot)
- [Cloudflare pages deploy and purge cache](#cloudflare-pages-deploy-and-purge-cache)

---

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
  workflow_dispatch:
  schedule:
    - cron: "30 7 * * 1-5"
  pull_request_target:
    types: [opened, synchronize, reopened]

jobs:
  auto-merge-dependabot:
    if: github.event_name != 'pull_request_target' || github.event.pull_request.user.login == 'dependabot[bot]'
    uses: manuelvanrijn/reusable_github_action_workflows/.github/workflows/auto-merge-dependabot.yml@main
    secrets:
      personal_access_token: ${{ secrets.PAT_TOKEN }}
```

---

### Cloudflare pages deploy and purge cache

This workflow deploys a project to Cloudflare Pages and optionally purges the cache for the specified zone.

#### Requirements:

##### Cloudflare API key

This action requires a [cloudflare api key](https://dash.cloudflare.com/profile/api-tokens) with the following scopes, to be able to deploy and clear the cache:

- Cloudflare Pages:Edit
- Cache Purge:Purge

##### Cloudflare id values

The `zone_id` and `account_id` can be found on the overview page of the domain.

#### Usage:

The following example demonstrates how to call this workflow:

```yaml
name: Build & Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-24.04
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@v4
      - run: mkdir -p build # implement build steps

      - name: Store build artifact
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: build
          overwrite: true
          retention-days: 1

  deploy:
    needs: build
    permissions:
      contents: read
      deployments: write
      pull-requests: write
    uses: manuelvanrijn/reusable_github_action_workflows/.github/workflows/cloudflare-pages-deploy-and-purge-cache.yml@main
    with:
      artifact_name: build
      cloudflare_project_name: "your-project-name"
      purge_cache: true
    secrets:
      cloudflare_api_token: ${{ secrets.CLOUDFLARE_API_TOKEN }}
      cloudflare_account_id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
      cloudflare_zone: ${{ secrets.CLOUDFLARE_ZONE_ID }}
      gh_token: ${{ secrets.GITHUB_TOKEN }}
```

> [!NOTE]
> The `purge_cache` input is optional and defaults to `true`. Set it to `false` if you do not want to purge the cache after deployment.
