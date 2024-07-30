# sync-workflows

The workflows are used to sync the forked repos between the GH orgs.

## sync-with-upstream.yml

Is needed to execute the sync between the upstream and the fork.

Inputs: see the "inputs" and "secrets" section of the code, they are related to a Github application(s) that are able to read the upstream and write the fork.

Omit the upstream code reader app part if you're sure that a standard GitHub token is OK (public repos case). Fork writing app can't be omitted.

The fork writer app must have permissions: `contents: write` and an undocumented permission for the upstream merge API `workflows: write`.

Behaviour of the workflow is also controlled by the repo/org _variable_ `BRANCHES_TO_SYNC_EGREP_REGEX` - it defines which upstream branches should be in the fork.

Must be called as a **job** with the `uses` directive.

Example:

```yml
jobs:
  sync_with_upstream:
    uses: some-gh-org/repo/.github/workflows/sync-with-upstream.yml@main
    with:
      fork_writer_app_id: ${{ vars.UPSTREAM_CONTENT_SYNC_APP_ID }}
      fork_owner: ${{ vars.RELEASE_ORG }}
    secrets:
      fork_writer_app_key: ${{ secrets.UPSTREAM_CONTENT_SYNC_APP_KEY }}
```

## check-syncronization.yml

Is needed to check the sync between the upstream and the fork.

Inputs: see the "inputs" and "secrets" section of the code, they are related to a Github application(s) that are able to read the upstream and and the fork. Omit them if you're sure that a standard GitHub token is OK (public repos case).

Output: `checks_passed`: Equal to 'true' if all of the conditions are OK:

- the upstream/fork refs, where the workflow is run, are really synced
- the workflow caller is the fork
- repo/org variable `RELEASES_ON` is `true`: can be used as an emergency killswitch to stop the release workflows globally (org level or per repo)

Must be called as a **job** with the `uses` directive.

Example:

```yml
jobs:
  check_syncronization:
    uses: some-gh-org/repo/.github/workflows/sync-with-upstream.yml@main
  
  some_release_job:
    needs: check_syncronization
    if: needs.check_syncronization.outputs.checks_passed == 'true'
    ...
```
