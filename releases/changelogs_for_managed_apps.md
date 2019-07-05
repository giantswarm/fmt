# Changelogs For Managed Apps

We want URLs showing changes between releases work for managed apps
repositories. That's why we push version tag pointing to the squashed commit in
the master branch that bumps the version. This unfortunately has to happen
after merge.

Example changelog in a good condition with working URLs can be found here:

https://github.com/giantswarm/kubernetes-node-exporter/blob/eb7b687f36463e1508c6f39614bec715bfe3bb58/CHANGELOG.md

Here is an URL taken from the example above showing changes between v4.0.0 and
v4.0.1:

https://github.com/giantswarm/kubernetes-node-exporter/compare/v0.4.0...v0.4.1

To tag a commit and push it:

```bash
git v1.2.3 GIT_SHA

git push --tags origin
```
