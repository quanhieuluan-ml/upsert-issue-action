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

| Input Name   | Required | Default Value | Description                                                                               | Type   |
| ------------ | -------- | ------------- | ----------------------------------------------------------------------------------------- | ------ |
| github-token | Yes      | N/A           | GitHub token for accessing the API                                                        | String |
| pattern      | Yes      | `**/*.sql`    | Glob pattern to match changed files                                                       | String |
| url          | Yes      | N/A           | The Bytebase URL. Example: https://bytebase.example.com                                   | String |
| token        | Yes      | N/A           | The API token obtained from bytebase/login action                                         | String |
| headers      | Yes      | N/A           | JSON string of extra headers to include in the request. e.g Cloudflare Zero Trust headers | String |
| project-id   | Yes      | N/A           | The project ID. Example: example                                                          | String |
| database     | Yes      | N/A           | The name of database. Example: instances/prod-instance/databases/example                  | String |
| title        | Yes      | N/A           | The title of the issue                                                                    | String |
| description  | Yes      | ``            | The description of the issue (can be empty)                                               | String |
| assignee     | Yes      | N/A           | The assignee of the issue                                                                 | String |
