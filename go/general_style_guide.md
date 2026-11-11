# Go Style Guide

> [Gofmt's style is no one's favorite, yet gofmt is everyone's favorite.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=8m43s)
>
> -- Rob Pike

It is important to follow this guide. It makes review process easier and
faster. It also makes our codebase consistent, easier to navigate, and
understand.

## Code Quality

We lint with [`golangci-lint`](https://golangci-lint.run/). `devctl gen makefile`
generates the repository's `.golangci.yaml`, and `make lint` runs it together
with the import check. CI runs `gosec` through
[`architect-orb`](https://github.com/giantswarm/architect-orb)'s `go-test`,
reading the same configuration.

### False Positives

When a finding is wrong for one place, silence it there with a
`//nolint:<linter> // <reason>` comment. When a rule does not fit the
repository, exclude it in `.golangci.yaml` under `linters.exclusions`, with a
comment that says why. See
[False Positives](https://golangci-lint.run/usage/false-positives/).

### Tooling

You can lint your code straight from the editor

- [IntelliJ](./intellij.md)

## Idiomatic Go

We write Go the way the Go project describes it in [Effective Go][effective-go]
and the [Go Code Review Comments][code-review-comments]. Where a guide in this
repository says something more specific, the guide in this repository applies.

[code-review-comments]: https://go.dev/wiki/CodeReviewComments
[effective-go]: https://go.dev/doc/effective_go

## Reuse before new code

Before we write a new function, type or helper, we check whether something
already does the job. We look in this order and use the first that fits:

1. The standard library.
2. A dependency the module already has in `go.mod`.
3. A helper in the repository.
4. New code.

A new dependency is the last option. It has to do something that would be
hard to write and maintain ourselves. This rule is about new code and new
dependencies: a repository keeps the libraries it already uses.

Three similar lines are fine. We extract a helper when the same logic is in
several places, not before.

## Vulnerability and Dependency Management

For keeping dependencies up to date, we use [Renovate](../github/renovate.md).

While Renovate helps to keep dependencies fresh, it is still possible for them to be vulnerable.
For that reason, we also use [`nancy`](https://github.com/sonatype-nexus-community/nancy) to scan a project's dependencies for known vulnerabilities at build time.

A finding from `nancy` might look like:

```shell
[1/1]	pkg:golang/github.com/gorilla/websocket@1.4.0
1 known vulnerabilities affecting installed version 
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ CWE-190: Integer Overflow or Wraparound                                                                                                                            ┃
┣━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃ Description        ┃ The software performs a calculation that can produce an integer overflow or                                                                   ┃
┃                    ┃ wraparound, when the logic assumes that the resulting value will always be                                                                    ┃
┃                    ┃ larger than the original value. This can introduce other weaknesses when                                                                      ┃
┃                    ┃ the calculation is used for resource management or execution control.                                                                         ┃
┣━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃ OSS Index ID       ┃ 5f259e63-3efb-4c47-b593-d175dca716b0                                                                                                          ┃
┣━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃ CVSS Score         ┃ 7.5/10 (High)                                                                                                                                 ┃
┣━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃ CVSS Vector        ┃ CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:N/A:H                                                                                                  ┃
┣━━━━━━━━━━━━━━━━━━━━╋━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
┃ Link for more info ┃ https://ossindex.sonatype.org/vuln/5f259e63-3efb-4c47-b593-d175dca716b0?component-type=golang&component-name=github.com%2Fgorilla%2Fwebsocket ┃
┗━━━━━━━━━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Summary                       ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━┫
┃ Audited Dependencies    ┃ 223 ┃
┣━━━━━━━━━━━━━━━━━━━━━━━━━╋━━━━━┫
┃ Vulnerable Dependencies ┃ 1   ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━┻━━━━━┛

```

For each identified vulnerability:

1. First, try to update the vulnerable dependency. If the dependency is not a _direct_ dependency, try using `go mod graph` to identify which package imports it, and update that instead. [helcaraxan/gomod](https://github.com/Helcaraxan/gomod) is also incredibly useful for debugging dependency issues. It can identify and draw paths to a specific dependency with `gomod graph 'rdeps(<your dependency>)'` and even represent the dependency tree graphically by piping that output to `dot`: `| dot -T png -o deps.png`. It is also available as a [docker image](https://hub.docker.com/r/amstaffix/gomod), including older versions like 0.5.0 which include the native ability to output a PNG (use the `-V -o <filename>.png` flags).
2. If the vulnerable dependency can't be resolved by updating a direct dependency, add a `replace` directive to your `go.mod` file.
For example, in the finding above, `gorilla/websocket v1.4.0` is vulnerable, but `v1.4.2` has been fixed.
This vulnerability can be mitigated by adding the following to the project's `go.mod` ():

 ```go
 replace github.com/gorilla/websocket v1.4.0 => github.com/gorilla/websocket v1.4.2
 ```

3. If the fixed dependency version introduces a breaking change or is otherwise impossible to update, you can suppress the finding temporarily to allow time for the dependency to be updated upstream.

When ignoring a dependency, always include a reasonable expiration date (as a suggestion, within 30 days).
Never create an ignore rule with no expiration -- every vulnerable dependency should either be updated or removed.
The suppression is only to unblock development until a patch is released.
If an update is still not available after the 30 day suppression period, this suggests that one of the components used is not regularly maintained. The dependency should be replaced with an alternative or `require`d by the project directly to ensure it continues to stay up to date.

To suppress a finding, add the name of the vulnerability to a file named `.nancy-ignore` in the root of your repository.

For example, this file would suppress the warning above until October 1st 2020:

```text
# .nancy-ignore

CWE-190 until=2020-10-01
```

Often, the vulnerability is referenced by a CVE number instead of a CWE.
You can suppress specific CVE numbers in the same way:

```text
# .nancy-ignore

CVE-2020-15114 until=2020-10-01
```

You can run `nancy` locally by building or downloading the binary and running `go list -json -m all | nancy -quiet` from the root of your project.

### Specifying CWE in a nancy-ignore file does not work

Instead of `CWE-xx`, use its OSS Index ID. You can find it in the nancy output.

```
7a182d40-9217-4391-85e9-6c44219efa7a until=2022-01-01 # CWE-79
3f9f4830-26e4-4acd-915b-448d38934e43 until=2022-01-01 # CWE-601
```

## Secrets

Please don't push secrets to GitHub unless they are encrypted.

We recommend [configuring a client-side git hook](https://github.com/zricethezav/gitleaks/wiki/Scanning#uncommitted-changes-scan) so that your secrets are never pushed in the first place. To do this:
```bash
# Create a directory to hold your global git hooks, if you don't have one
$ mkdir /path/to/your/git-hooks

# Change to the directory
$ cd /path/to/your/git-hooks

# Create a file named pre-commit
$ touch pre-commit

# Add the git hook content from the gitleaks wiki
$ cat <<EOS >> pre-commit
#!/bin/sh
# Adds gitleaks as a pre-commit hook.

gitleaksEnabled=\$(git config --bool hooks.gitleaks)
cmd="gitleaks --verbose --redact --pretty"
if [ "\$gitleaksEnabled" = "true" ]; then
    \$cmd
    if [ \$? -eq 1 ]; then
cat <<\EOF
Error: gitleaks has detected sensitive information in your changes.
If you know what you are doing you can disable this check using:
    git config hooks.gitleaks false
EOF
exit 1
    fi
fi
EOS

# Add the path as your global git hook location
$ git config --global core.hooksPath /path/to/your/git-hooks

# Enable gitleaks globally
$ git config --global hooks.gitleaks true
```

Just in case someone accidentally pushes a secret, there is a [`gitleaks`](https://github.com/zricethezav/gitleaks/) workflow included in `devctl gen workflows` which will scan your commits and fail your build if a suspected secret is found.

By default, it will scan your PR branch back to the latest ancestor with the target branch.
If there is no ancestor, or if you are committing directly to `master`, the whole branch history will be scanned, which can take some time on large projects.

Once a secret is pushed to a public repository, however, it must be rotated.

### Overriding False Positives

Secret scanning is currently more art than science.
`gitleaks` does its best, but it is possible you will encounter false positives (especially when dealing with high-entropy strings).

If that happens, you can customize the scanning behavior by including a `.github/.gitleaks.toml` file in your repository.
This file allows you to specify particular regular expressions or paths to ignore when scanning.
To retain as much coverage as possible, we suggest matching your excluded value as exactly as possible (e.g. if you have a non-secret token to check in, simply exclude a regex containing the token value).

You can add new rules or override the default configuration through the `.github/.gitleaks.toml` file.

There are [a few ways to exclude something](https://github.com/zricethezav/gitleaks/wiki/Configuration), but here are some quick examples:

```toml
# in .gitleaks.toml

# ...
# lots of default rules here
# ...

[allowlist]
    # Default gitleaks exclusions
    description = "Allowlisted files"
    files = ['''^\.?gitleaks.toml$''',
    '''(.*?)(jpg|gif|doc|pdf|bin)$''',
    '''(go.mod|go.sum)$''']

    # Exclude anything in vendor/
    paths = [ '''vendor\/.*''']

    # Exclude a certain value
    regexes = ['''MyPublicOkToCheckInTokenValue''']

    # Exclude a file by adding it to the existing default 'files' list, e.g.:
    files = ['''^this-file-is-all-high-entropy-strings.json$''',
        '''^\.?gitleaks.toml$''',
        '''(.*?)(jpg|gif|doc|pdf|bin)$''',
        '''(go.mod|go.sum)$''']
```

### Disabling for a Repository

If it is simply too noisy to use the scanner on a particular repository, it is possible to exclude the workflow.

If your repository is managed by [`giantswarm/github`](https://github.com/giantswarm/github/), you can set this via the `gen` options in `meta.yaml`:

```yaml
- name: my-operator
  gen:
    flavour: app
    checkSecrets: false
  team: my-team
```
