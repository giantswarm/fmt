# Branches

This page describes branch conventions we follow.

## Release long-lived branches

We keep long-lived branches for (usually major, occasionally minor) release
lines we still maintain. Here are the rules we apply to those branches:

- Branch names have format `release-v1.x.x` or `release-v1.2.x`.
- Release tags in format `v1.2.3` should be applied only to commits in those
  branches and the `master` branch.
- They should be created immediately after new major release for the previous
  major line. E.g. when `v3.0.0` is release we should have `release-v2.x.x`
  branch.
- Once we stop maintaining the release line the branch should be removed from
  GitHub.
- We strongly prefer to not maintain minor release lines. I.e. branches with
  names like `release-v1.2.x`. If one is created there needs to be a good
  reason for it.
