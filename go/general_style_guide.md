# Go Style Guide

> [Gofmt's style is no one's favorite, yet gofmt is everyone's favorite.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=8m43s)
>
> -- Rob Pike

It is important to follow this guide. It makes review process easier and
faster. It also makes our codebase consistent, easier to navigate, and
understand.

## Code Quality

To prevent shipping problematic or unsafe code, [`architect-orb`](https://github.com/giantswarm/architect-orb)'s
[`go-test`](https://github.com/giantswarm/architect-orb/blob/master/src/commands/go-test.yaml) and
[`go-test-legacy`](https://github.com/giantswarm/architect-orb/blob/master/src/commands/go-test-legacy.yaml)
commands include running several linting tools via [`golangci-lint`](https://github.com/golangci/golangci-lint).

You can install and run this tool locally to lint your code before committing. Follow the instructions in the `golangci-lint` docs.

The test (at the time of this writing) runs all default linters, plus `gosec`, `goconst`, and `unparam`.

You can run this configuration locally by using `golangci-lint run -E gosec -E goconst -E unparam`

### False Positives

The linter's job is to enforce conformity, but if you find that one of the rules conflicts with Giant Swarm style or functional needs,
it is possible to exclude the offending rule.

1. For a specific rule which should be suppressed globally,
it can be excluded with `-e < specific regex to match the error>`
2. For a folder or file path which should never be linted (globally),
it can be excluded with `--skip-dirs <regex to match directories>`
or `--skip-files <regex to match files>`
3. For a rule which should be suppressed only for a specific code line or block,
it can be silenced in the code with `//no-lint:<linter name>`.
This should be used sparingly to avoid forming a bad habit of simply silencing bad code.

**Note:** Options 1 and 2 can only be applied by updating the command in `architect-orb`.

Details for using all of these methods can be found in the
[False Positives](https://github.com/golangci/golangci-lint#false-positives) section of the docs.

### Tooling

You can lint your code straight from the editor

- [Intellij](./intellij.md)

## Vulnerability and Dependency Management

For keeping dependencies up to date, we use [Dependabot](../github/dependabot), which is documented separately.

While Dependabot helps to keep dependencies fresh, it is still possible for them to be vulnerable.
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

1. First, try to update the vulnerable dependency. If the dependency is not a _direct_ dependency, try using `go mod graph` to identify which package imports it, and update that instead.
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

## Secrets

Please don't push secrets to GitHub unless they are encrypted.

We recommend [configuring a client-side git hook](https://github.com/zricethezav/gitleaks/wiki/Scanning#uncommitted-changes-scan) so that your secrets are never pushed in the first place.

Just in case someone accidentally pushes a secret, there is a [`gitleaks`](https://github.com/zricethezav/gitleaks/) workflow included in `devctl gen workflows` which will scan your commits and fail your build if a suspected secret is found.

By default, it will scan your PR branch back to the latest ancestor with the target branch.
If there is no ancestor, or if you are committing directly to `master`, the whole branch history will be scanned, which can take some time on large projects.

Once a secret is pushed to a public repository, however, it must be rotated.

### Overriding False Positives

Secret scanning is currently more art than science.
`gitleaks` does its best, but it is possible you will encounter false positives (especially when dealing with high-entropy strings).

If that happens, you can customize the scanning behavior by including a `.gitleaks.toml` file in your repository.
This file allows you to specify particular regular expressions or paths to ignore when scanning.
To retain as much coverage as possible, we suggest matching your excluded value as exactly as possible (e.g. if you have a non-secret token to check in, simply exclude a regex containing the token value).

Unfortunately, it isn't currently possible to provide a "patch" to the default configuration (this capability has been requested in [this issue](https://github.com/zricethezav/gitleaks/issues/429)).
If you include a `.gitleaks.toml` file, you must use the [default configuration](https://github.com/zricethezav/gitleaks/blob/master/config/default.go) as a base, and apply your exclusion(s) on top of that.
There are [a few ways to exclude something](https://github.com/zricethezav/gitleaks/wiki/Configuration), but here are some quick examples:

```toml
# in .gitleaks.toml

...
# lots of default rules here
...

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
    files = ['''^this-file-is-all-high-entropy-strings.json$'''
        '''^\.?gitleaks.toml$''',
        '''(.*?)(jpg|gif|doc|pdf|bin)$''',
        '''(go.mod|go.sum)$''']
```

### Disabling for a Repository

If it is simply too noisy to use the scanner on a particular repository, it is possible to exclude the workflow.

If you are generating workflows with `devctl`, you can use the `--check-secrets` flag to control the workflow:

```shell
devctl gen workflows --flavour <flavour> --check-secrets=false
```

If your repository is managed by [`giantswarm/github`](https://github.com/giantswarm/github/), you can set this via the `gen` options in `meta.yaml`:

```yaml
- name: my-operator
  gen:
    flavour: app
    checkSecrets: false
  team: my-team
```
