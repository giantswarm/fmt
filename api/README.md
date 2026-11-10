# Specifications

We build on protocols and API styles that other people have specified. We
follow those specifications, so that our APIs behave the way clients, SDKs and
other engineers already expect. This guide says which specification applies to
which kind of interface, what to do when a change would not follow it, and how
each rule is checked.

- [Which specification applies](#which-specification-applies)
- [When two conventions meet](#when-two-conventions-meet)
- [Deviating](#deviating)
- [MCP](#mcp)
- [How each rule is checked](#how-each-rule-is-checked)

## Which specification applies

| What we build | Specification | Guide |
|---|---|---|
| REST and gRPC APIs for our own resources | [Google AIPs][aip] | [REST APIs](rest.md) |
| MCP servers and clients | [Model Context Protocol][mcp], the revision the repository pins | [MCP](#mcp) |
| MCP tool arguments and results that list or change our resources | [Google AIPs][aip] | [REST APIs](rest.md) |
| A2A servers and clients | [Agent2Agent protocol][a2a] | |
| Authorization and identity | OAuth 2.1, OpenID Connect, RFC 8693 | [OAuth and OIDC](../security/oauth.md) |
| Kubernetes custom resources | [Kubernetes API conventions][k8s-api] | [Annotations and Labels](../kubernetes/annotations_and_labels.md) |
| Code inside a fork or a contribution to another project | That project's own conventions | |

Names are part of the specification. See [Names](names.md).

## When two conventions meet

The specification of the protocol we implement wins over a convention from
somewhere else.

Pagination is the common case. Our REST APIs follow [AIP-158][aip-158], where
an empty `nextPageToken` means the end of the collection. MCP list methods
(`tools/list`, `resources/list`, `prompts/list`) follow the
[MCP pagination rules][mcp-pagination], where the fields are `cursor` and
`nextCursor`, and where:

> an empty string is a valid cursor and thus **MUST NOT** be treated as the end
> of results.

So an MCP server uses `nextCursor` for the protocol's own lists, and
`pageToken` and `nextPageToken` inside a tool that lists our resources.

## Deviating

Sometimes following the specification is not possible or not worth it. Then:

- Ask before writing the change. An agent asks the user. An engineer asks the
  team that owns the interface.
- Say in the pull request what deviates, from which section, and why.
- When a linter checks the rule, switch it off for that one place, inline,
  with the reason next to it. We do not switch a rule off for the whole
  repository to get one case through.
- A deviation that stays is written down in the repository's `AGENTS.md`,
  in the section for that repository, so the next person does not "fix" it.
- A deviation that only exists in our fork links the upstream issue or pull
  request that removes it.

**Don't do this**: add a `truncated: true` field to a list response because
it is quicker than paging, and mention it nowhere.

## MCP

- We pin the specification revision we implement and name it in the
  repository's documentation.
- Lists of the protocol use `cursor` and `nextCursor`, as described above.
- We do not pass tokens through. The [authorization specification][mcp-security]
  says:

  > MCP servers **MUST** only accept tokens specifically intended for
  > themselves

  and

  > The MCP server **MUST NOT** pass through the token it received from the
  > MCP client.

  A server that calls another API gets a separate token for that API. See
  [OAuth and OIDC](../security/oauth.md).
- The [MCP conformance suite][mcp-conformance] tests a server or a client
  against the specification. We pin its version and keep the list of expected
  failures in the repository.

## How each rule is checked

A rule that a tool checks is worth more than one that is only written down.
For each rule, this table names the strongest check we have. "Review" means
a person or the `review-pr` skill of the `giantswarm/claude-code` plugins.

| Rule | Source | Check |
|---|---|---|
| Follow the specification of the protocol | the specification | review |
| Ask before deviating, say why in the pull request | this guide | review |
| Resource names, standard methods, field names in REST APIs | AIP-122, AIP-131 to AIP-135, AIP-140 | OpenAPI linter in CI |
| Pagination fields, no `truncated` flag | AIP-158 | OpenAPI linter in CI |
| Pagination behaviour (end of collection, page size) | AIP-158 | test in the repository |
| Error response shape | AIP-193 | OpenAPI linter in CI |
| Error codes and details on real responses | AIP-193 | test in the repository |
| MCP protocol behaviour | MCP specification | MCP conformance suite in CI |
| No token passthrough, audience validation | MCP authorization | test in the repository, review |
| OAuth 2.1, token exchange | [OAuth and OIDC](../security/oauth.md) | test in the repository, review |
| Names come from the specification or the domain | [Names](names.md) | review |
| Reuse before new code | [Go Style Guide](../go/general_style_guide.md#reuse-before-new-code) | review |
| Actions and images are pinned | OpenSSF Scorecard | zizmor and Scorecard workflows |

The OpenAPI linter and the MCP conformance suite are being introduced
repository by repository. Until a repository runs them, review is the check.

[a2a]: https://a2a-protocol.org/latest/specification/
[aip]: https://google.aip.dev/
[aip-158]: https://google.aip.dev/158
[k8s-api]: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md
[mcp]: https://modelcontextprotocol.io/specification/latest
[mcp-conformance]: https://github.com/modelcontextprotocol/conformance
[mcp-pagination]: https://modelcontextprotocol.io/specification/2026-07-28/server/utilities/pagination
[mcp-security]: https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization/security-considerations
