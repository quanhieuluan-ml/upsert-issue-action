# About

Github action to upsert issues on Bytebase.

This action creates or updates the Bytebase migration issue for the PR. If you
change the migration script during the PR process, this action will update the
corresponding Bytebase migration task as well. And it will return error if you
attempt to update a migration script when the corresponding migration task has
already been rolled out.

# Change Type

The migration files matched by `pattern` are assumed to follow a naming scheme.
It should start with digit version numbers, separated by underscore and end with
a change type. Example: `path/to/migrations/<<version>>_xxxx_<<changeType>>.sql`
This action determines the change type of a file by how its filename ends. The
default change type is schema change.

| Ends with | Type                 |
| --------- | -------------------- |
| ddl       | schema change        |
| ghost     | online schema change |
| dml       | data change          |

# Inputs

| Input Name   | Required | Default Value | Description                                                                                                                                                                                                                                                                                                                                                                                          | Type   |
| ------------ | -------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| github-token | Yes      | N/A           | GitHub token for accessing the API                                                                                                                                                                                                                                                                                                                                                                   | String |
| pattern      | Yes      | `**/*.sql`    | Glob pattern to match changed files                                                                                                                                                                                                                                                                                                                                                                  | String |
| url          | Yes      | N/A           | The Bytebase URL. Example: https://bytebase.example.com                                                                                                                                                                                                                                                                                                                                              | String |
| token        | Yes      | N/A           | The API token obtained from bytebase/login action                                                                                                                                                                                                                                                                                                                                                    | String |
| headers      | Yes      | N/A           | JSON string of extra headers to include in the request. e.g Cloudflare Zero Trust headers                                                                                                                                                                                                                                                                                                            | String |
| project-id   | Yes      | N/A           | The project ID. Example: example                                                                                                                                                                                                                                                                                                                                                                     | String |
| targets      | Yes      | N/A           | The database group or databases to deploy. Either a comma separated list of the databases or a database group. Databases example: `instances/mysql1/databases/db1,instances/mysql1/databases/db2`. Database format: instances/{instance}/databases/{database} Database group example: `projects/exa/databaseGroups/mygroup` Database group format: projects/{project}/databaseGroups/{databaseGroup} | String |
| title        | Yes      | N/A           | The title of the issue                                                                                                                                                                                                                                                                                                                                                                               | String |
| description  | Yes      | ``            | The description of the issue (can be empty)                                                                                                                                                                                                                                                                                                                                                          | String |

## Example

```yaml
name: Upsert Migration

on:
  pull_request:
    branches:
      - main
    types:
      - closed # Triggers when a pull request is closed

jobs:
  bytebase-upsert-migration:
    runs-on: ubuntu-latest
    # Runs only if PR is approved and target branch is main
    if:
      github.event.review.state == 'approved' &&
      github.event.pull_request.base.ref == 'main'
    env:
      BYTEBASE_URL: 'https://bytebase-ci.zeabur.app'
      BYTEBASE_SERVICE_ACCOUNT: 'ci@service.bytebase.com'
      PROJECT: 'example'
      TARGETS: 'instances/prod-instance/databases/example'
      ISSUE_TITLE:
        '[${{ github.repository }}#${{ github.event.pull_request.number }}] ${{
        github.event.pull_request.title }}'
      DESCRIPTION:
        'Triggered by ${{ github.event.repository.html_url }}/pull/${{
        github.event.pull_request.number }} ${{ github.event.pull_request.title
        }}'
    name: Upsert Migration
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Login to Bytebase
        id: login
        uses: bytebase/login-action@0.0.2
        with:
          bytebase-url: ${{ env.BYTEBASE_URL }}
          service-key: ${{ env.BYTEBASE_SERVICE_ACCOUNT }}
          service-secret: ${{ secrets.BYTEBASE_PASSWORD }}
      - name: Upsert issue
        id: upsert
        uses: bytebase/upsert-issue-action@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          pattern: '**/*.sql'
          url: ${{ env.BYTEBASE_URL }}
          token: ${{ steps.login.outputs.token }}
          headers: '{"Accept-Encoding": "deflate, gzip"}'
          project-id: ${{ env.PROJECT }}
          targets: ${{ env.TARGETS }}
          title: ${{ env.ISSUE_TITLE }}
          description: ${{ env.DESCRIPTION }}
```
