# sync-workflows

The workflows are used to sync the forked repos between the GH orgs.

## sync-with-upstream.yml

It is needed to execute the controlled sync between the upstream and the fork.

Inputs: see the "inputs" and "secrets" section of the code, they are related to a Github application(s) that are able to read the upstream and write the fork.

A specific input is `delete_non_matching_refs`, boolean (no quotes), default is `false`. When is set to `true`, it means that the refs (branches/tags), which exist in the fork, but do not exist in the upstream, will be wiped.

Omit the upstream code reader app part if you're sure that a standard GitHub token is OK (a public repos case). Fork writing app can't be omitted because of the undocumented Github API limitations, related to the `workflows: write` permission.

The fork writer app must have the permissions: `contents: write` and the undocumented permission for the upstream merge API `workflows: write`.

Behaviour of the workflow is also controlled by the repo/org _variable_ `BRANCHES_TO_SYNC_EGREP_REGEX` - it defines which upstream branches should be in the fork. By default all the branches will be synced (not really good).

Must be called as a **job** with the `uses` directive. Example:

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

This action should prepend all release workflows.

It is needed to check the sync between the upstream and the fork, and also if the workflow is executed from the release org. It checks if the calling ref (branch or tag) has the same commit hash in the upstream and the fork.

When the refs are in sync, it returns `true`. If the target ref is missing in the upstream (i.e. was created in the fork only), the workflow returns `true` as well.

Inputs: see the "inputs" and "secrets" section of the code, they are related to a Github application(s) that are able to read the upstream and and the fork. Omit them if you're sure that a standard GitHub token is OK (a public repos case).

Output: `checks_passed`: Equal to 'true' if all of the conditions are OK:

- the upstream/fork refs, where the workflow is run, are really synced
- the workflow caller is the fork
- repo/org variable `RELEASES_ON` is `true`: can be used as an emergency killswitch to stop the release workflows globally (org level or per repo)

Must be called as a **job** with the `uses` directive. Example:

```yml
jobs:
  check_syncronization:
    uses: some-gh-org/repo/.github/workflows/sync-with-upstream.yml@main
  
  some_release_job:
    needs: check_syncronization
    if: needs.check_syncronization.outputs.checks_passed == 'true'
    ...
```
