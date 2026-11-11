# Renovate

[Renovate](https://docs.renovatebot.com/) opens pull requests that update our
dependencies: Go modules, Helm chart dependencies, container images and GitHub
Actions. Staying current is how most vulnerability fixes reach our code.

Most repositories get their `renovate.json5` from `devctl`, generated from the
repository's entry in the team file of `giantswarm/github`. Where it is
generated, change the entry rather than the file. An entry with
`renovate: false` gets no configuration.
