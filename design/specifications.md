# Specifications

Most of what we build implements something other people have specified: the
Kubernetes API, a Helm chart, a metric, an MCP server, an OAuth flow. We follow
that specification, so our code behaves the way users, clients and other
engineers already expect.

- [Which specification applies](#which-specification-applies)
- [When two conventions meet](#when-two-conventions-meet)
- [Deviating](#deviating)
- [How the rules are checked](#how-the-rules-are-checked)

## Which specification applies

| What we build | Specification | Our guide |
|---|---|---|
| Custom resources and controllers | [Kubernetes API conventions][k8s-api] | [Annotations and Labels](../kubernetes/annotations_and_labels.md), [Idempotency](../go/idempotency_conventions.md) |
| Cluster API providers | [Cluster API provider contracts][capi-contracts] | |
| Helm charts | [Helm chart best practices][helm] | [Helm](../README.md#helm-charts) |
| Prometheus metrics and rules | [Prometheus naming][prom-naming] | [Prometheus](../README.md#prometheus) |
| Traces, logs and OpenTelemetry metrics | [OpenTelemetry semantic conventions][semconv] | |
| HTTP APIs | [HTTP semantics (RFC 9110)][rfc-9110] | [HTTP APIs](http_apis.md) |
| MCP servers and clients | [Model Context Protocol][mcp], the revision the repository pins | |
| A2A servers and clients | [Agent2Agent protocol][a2a] | |
| Authorization and sign-in | OAuth 2.1, OpenID Connect | [OAuth and OIDC](../security/oauth.md) |
| A fork, or a contribution to another project | That project's own conventions | |

Names are part of the specification. See [Names](names.md).

## When two conventions meet

- The specification of the protocol we implement wins over a convention from
  somewhere else.
- A change to an existing API follows the style that API already has, even
  where a guide here would choose differently. Consistency inside one API
  matters more to its clients. A new API, or a new major version, follows the
  guides.

## Deviating

Sometimes following the specification is not possible or not worth it. Then:

- Ask before writing the change. An engineer asks the team that owns the
  interface. An agent asks the user; when nobody can be asked, it states the
  deviation in a draft pull request and waits for review.
- Say in the pull request what deviates, from which section, and why.
- When a linter checks the rule, switch it off inline for that one place, with
  the reason next to it, not for the whole repository.
- A deviation that stays is written down in the repository's `AGENTS.md`, so
  the next person does not "fix" it.
- A deviation that only exists in our fork links the upstream issue or pull
  request that removes it.

**Don't do this**: add a `truncated: true` field to a list response because it
is quicker than paging, and mention it nowhere.

## How the rules are checked

A rule that a tool checks is worth more than one that is only written down.
Where a check exists, the repository runs it in CI:

- Linters that already check a specification: `golangci-lint`, `helm lint`,
  `promtool check rules`, [kube-api-linter][kal] for Kubernetes API types.
- The [MCP conformance suite][mcp-conformance] for MCP servers and clients,
  with its version pinned and the expected failures listed in the repository.
- Tests in the repository for behaviour no linter sees: pagination, error
  responses, the tokens an API accepts and refuses.

Everything else is checked in review, by a person or by the `review-pr` skill
of the `giantswarm/claude-code` plugins.

[a2a]: https://a2a-protocol.org/latest/specification/
[capi-contracts]: https://cluster-api.sigs.k8s.io/developer/providers/contracts/overview
[helm]: https://helm.sh/docs/chart_best_practices/
[kal]: https://github.com/kubernetes-sigs/kube-api-linter
[k8s-api]: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md
[mcp]: https://modelcontextprotocol.io/specification/latest
[mcp-conformance]: https://github.com/modelcontextprotocol/conformance
[prom-naming]: https://prometheus.io/docs/practices/naming/
[rfc-9110]: https://www.rfc-editor.org/rfc/rfc9110
[semconv]: https://opentelemetry.io/docs/specs/semconv/
