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

The linter's job is to enforce conformity, but if you find that one of the rules conflicts with Giant Swarm style or functional needs, it is possible to exclude the offending rule.

1. For a specific rule which should be suppressed globally, it can be excluded with `-e < specific regex to match the error>`
2. For a folder or file path which should never be linted (globally), it can be excluded with `--skip-dirs <regex to match directories>`
or `--skip-files <regex to match files>`
3. For a rule which should be suppressed only for a specific code line or block, it can be silenced in the code with `//no-lint:<linter name>`.
This should be used sparingly to avoid forming a bad habit of simply silencing bad code.

Options 1 and 2 can only be applied by updating the command in `architect-orb`.

Details for using all of these methods can be found in the [False Positives](https://github.com/golangci/golangci-lint#false-positives) section of the docs.
