# Dependabot

Dependabot is a service integrated into GitHub which watches the listed dependencies in
our repos and automatically opens PRs to update them when updates are available.

It will open a Pull Request and provide a Changelog and compatibility score for the update.
You can specify default reviewers for a given repository, apply labels automatically,
and Dependabot will rebase, update, or close the PR as necessary if the project's requirements
or the upstream updates change.

This should be enabled for every project by default and configured as necessary to reduce noise.

## Why

Some dependencies we manage ourselves, but not the vast majority.
Having automated PRs is helpful because it saves us the trouble of manually tracking updates
or maintaining a homegrown tool to do so.

However, this is more than a convenience feature.
Dependencies are a major way vulnerabilities are introduced in code, so staying up to date
is an important part of securing our platform.

For instance, [here are some vulnerabilities found in the k8s.io libraries we use](https://snyk.io/vuln/search?type=golang&q=k8s.io)

## Configuration

There are two ways to configure Dependabot:

1. Through [the web UI](https://app.dependabot.com/accounts/giantswarm/)
2. Using [configuration files](https://dependabot.com/docs/config-file/) (preferred)

Configuration files are preferred because they can be more easily managed per-project and per-branch directly in the repository,
and expose more configuration options than the web UI.

You can validate a Dependabot config file [using this validator](https://dependabot.com/docs/config-file/validator/)

A sample configuration file is included below, which excludes all `k8s.io` dependencies above `0.17.0`:

```yml
# .dependabot/config.yml
version: 1
update_configs:
  - package_manager: "go:modules"
    directory: "/"
    update_schedule: "daily"
    ignored_updates:
      - match:
          dependency_name: "k8s.io/*"
          version_requirement: ">=0.17.0"
```

When possible, exclude only known-breaking version changes (e.g. exclude only major versions).
This way, we continue to get updates which are compatible, but with less noise.
