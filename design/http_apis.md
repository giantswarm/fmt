# HTTP APIs

For an HTTP API that no other specification governs. A Kubernetes API, an MCP
server or a Backstage plugin follows its own specification first; see
[Which specification applies](specifications.md#which-specification-applies).
A change to an existing API follows the style that API already has.

- [Basics](#basics)
- [Pagination](#pagination)
- [Errors](#errors)

## Basics

- Methods and status codes mean what [RFC 9110][rfc-9110] says they mean.
- The API is described by an [OpenAPI][openapi] document in the repository.
- JSON field names are lowerCamelCase.

## Pagination

Every method that returns a collection is paginated from its first version.
Adding pagination later breaks clients.

The request has `pageSize` and `pageToken`, both optional. The response has the
items and `nextPageToken`.

- When `pageSize` is missing or `0`, the API picks a default. When it is larger
  than the maximum, the API returns at most the maximum. Neither is an error.
- An empty or missing `nextPageToken` is the only signal for the end of the
  collection. A page can hold fewer items than `pageSize` and still have a next
  page.
- Page tokens are opaque. Clients do not parse them, and a token does not grant
  access to anything.

**Don't do this**:

```json
{
  "agents": [],
  "truncated": true
}
```

A `truncated`, `hasMore` or `isLast` flag tells the client that data is missing
without giving it a way to get the rest.

**Do this**:

```json
{
  "agents": [],
  "nextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
}
```

## Errors

An error response is a problem details object as in [RFC 9457][rfc-9457], with
the media type `application/problem+json`:

```json
{
  "type": "https://agents.giantswarm.io/problems/agent-not-found",
  "title": "Agent not found",
  "status": 404,
  "detail": "Agent \"support-bot\" was not found.",
  "agent": "support-bot"
}
```

- `type` identifies the kind of error. Clients match on `type`, never on
  `title` or `detail`, and one `type` always means the same error.
- `status` is the HTTP status code of the response.
- Extra fields, such as `agent` above, carry what the client needs to act on
  the error.

[openapi]: https://spec.openapis.org/oas/v3.1.1.html
[rfc-9110]: https://www.rfc-editor.org/rfc/rfc9110
[rfc-9457]: https://www.rfc-editor.org/rfc/rfc9457
