# About

Github action to upsert issues on Bytebase.

This action creates or updates the Bytebase migration issue for the PR. If you
change the migration script during the PR process, this action will update the
corresponding Bytebase migration task as well. And it will return error if you
attempt to update a migration script when the corresponding migration task has
already been rolled out.
