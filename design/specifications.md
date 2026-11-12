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
| MCP servers and clients | [Model Context Protocol][mcp], the latest revision the SDK supports; the repository names the revision it implements | |
| A2A servers and clients | [Agent2Agent protocol][a2a] | |
| Authorization and sign-in | OAuth 2.1, OpenID Connect | [OAuth and OIDC](../security/oauth.md) |
| A fork, or a contribution to another project | That project's own conventions | |

In a custom resource, a true or false observation about the resource is a
`metav1.Condition` in `status.conditions`, not a `bool` field.

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
Each repository runs its checks through `make lint` and its tests, so that
pre-commit, agent hooks and CI run the same thing. Not every repository runs
every check yet.

| Rule | Guide | Check |
|---|---|---|
| Kubernetes API types follow the API conventions | this guide | [kube-api-linter][kal] |
| No new `bool` status fields; observations are conditions | this guide | kube-api-linter `nobools`, then review |
| Helm charts | [Helm](../README.md#helm-charts) | `helm lint`, the values schema |
| Prometheus rules | [Prometheus](../README.md#prometheus) | `promtool check rules` |
| OpenTelemetry metric and span names | this guide | [Weaver][weaver] live check, where the repository runs it |
| MCP protocol and authorization | this guide | [MCP conformance suite][mcp-conformance], version pinned, expected failures listed in the repository |
| Lists are paginated, no `truncated` flag | [HTTP APIs](http_apis.md) | a test in the repository; an OpenAPI linter for HTTP APIs |
| HTTP errors are RFC 9457 problem details | [HTTP APIs](http_apis.md) | an OpenAPI linter rule |
| Tokens refused for the wrong audience, issuer or expiry; no passthrough; token exchange | [OAuth and OIDC](../security/oauth.md) | tests in the repository, MCP conformance authorization scenarios |
| Names come from the specification, one meaning per word | [Names](names.md) | review |
| Which specification applies, an existing API's style, deviations stated | this guide | review |
| Reuse before new code | [Go Style Guide](../go/general_style_guide.md#reuse-before-new-code) | review |

Review is done by a person or by the `review-pr` skill of the
`giantswarm/claude-code` plugins.

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
[weaver]: https://github.com/open-telemetry/weaver
