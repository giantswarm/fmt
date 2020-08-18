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

The best course of action is to update the dependency.
If the dependency is nested, try using a `replace` directive in `go.mod` to use a safe version.
If it is not possible to update the dependency for technical or compatibility reasons, you can suppress the finding by adding it to a file named `.nancy-ignore` in the root of your repository.

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

## Tooling

You can lint your code straight from the editor
- [Intellij](./intellij.md)
