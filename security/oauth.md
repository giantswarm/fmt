# OAuth and OIDC

We implement authorization and sign-in with OAuth and OpenID Connect as
specified. Small departures in this area turn into security bugs, so the
rule to [ask before deviating](../design/specifications.md#deviating) matters most here.
We also do not design our own token formats or flows where a specified one
exists.

- [Baseline](#baseline)
- [Tokens are for one audience](#tokens-are-for-one-audience)
- [Acting for someone else](#acting-for-someone-else)
- [How this is tested](#how-this-is-tested)

## Baseline

We follow [OAuth 2.1][oauth-2-1] and the
[OAuth security best current practice][rfc-9700].

- Authorization code flow with PKCE. Only the `S256` method is accepted.
- No implicit flow and no password flow.
- Redirect URIs are compared as exact strings.

MCP servers and clients also do the following, because the MCP authorization
specification requires it. Other APIs do it when their clients and
authorization server support it:

- The API publishes [OAuth 2.0 Protected Resource Metadata][rfc-9728] at
  `/.well-known/oauth-protected-resource`, so that clients find its
  authorization server.
- Clients send the `resource` parameter of [RFC 8707][rfc-8707] in the
  authorization request and in the token request, so that the token is issued
  for one API.

## Tokens are for one audience

An API accepts a token only when the token was issued for it. It checks the
issuer, the signature, the expiry and the audience, and rejects the token when
any of them does not match.

An API does not send the token it received to another API. This is called
token passthrough, and the MCP specification forbids it. The other API would
get a token that was not issued for it, and could not tell who is calling.

**Don't do this**: copy the `Authorization` header of the incoming request to
an outgoing request.

**Do this**: get a new token for the other API, with that API as the audience.
See the next section.

## Acting for someone else

When a service calls another API for a person, it gets a token for that API
with [OAuth 2.0 Token Exchange][rfc-8693]. We use its terms and keep the two
parties apart:

| Term | Meaning | Section |
|---|---|---|
| `subject_token` | the token of the party the request is made for | 2.1 |
| `actor_token` | the token of the party that is acting | 2.1 |
| `act` claim | in the issued token, who is acting for the subject | 4.1 |
| `may_act` claim | who is allowed to act for the subject | 4.4 |

- The issued token keeps the person as the subject and names the service in
  `act`. The service does not become the subject.
- When no rule says that this actor may act for this subject, the exchange is
  refused.

## How this is tested

- Each service that accepts tokens has tests for the cases it must refuse:
  wrong audience, wrong issuer, expired, bad signature.
- Each service that calls another API has a test that the outgoing request
  does not carry the incoming token.
- Token exchange has tests for the subject and the actor of the issued token,
  and for the refused case.
- The [MCP conformance suite][mcp-conformance] has an authorization suite for
  MCP servers and clients.
- The [OpenID Foundation conformance suite][oidf] tests OpenID Connect
  providers and clients. It does not test OAuth 2.1 or token exchange, so our
  own tests cover those.

[mcp-conformance]: https://github.com/modelcontextprotocol/conformance
[oauth-2-1]: https://datatracker.ietf.org/doc/draft-ietf-oauth-v2-1/
[oidf]: https://gitlab.com/openid/conformance-suite
[rfc-8693]: https://www.rfc-editor.org/rfc/rfc8693
[rfc-8707]: https://www.rfc-editor.org/rfc/rfc8707
[rfc-9700]: https://www.rfc-editor.org/rfc/rfc9700
[rfc-9728]: https://www.rfc-editor.org/rfc/rfc9728
