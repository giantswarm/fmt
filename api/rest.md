# REST APIs

Our REST APIs follow Google's [API Improvement Proposals][aip] (AIPs). They
are a complete, public set of rules for resource-oriented APIs, and many
clients and engineers already know them. This guide lists the AIPs we rely on
most. The AIP itself is the reference: when this guide and the AIP differ, the
AIP is right and this guide needs a fix.

The same rules apply to gRPC APIs we define, and to MCP tools that list or
change our resources. For what to do when a rule cannot be followed, see
[Deviating](README.md#deviating).

- [Resource names](#resource-names)
- [Standard methods](#standard-methods)
- [Field names](#field-names)
- [Pagination](#pagination)
- [Errors](#errors)

## Resource names

[AIP-122][aip-122]. A resource has one name, unique within the API. The name
is a path that alternates collection and identifier, with `/` between the
segments:

```text
organizations/acme/agents/support-bot
```

Collection segments are plural and in camelCase.

## Standard methods

[AIP-131][aip-131] to [AIP-135][aip-135]. A resource is read and changed with
the standard methods before we invent a custom one.

| Method | HTTP | Returns |
|---|---|---|
| Get | `GET /v1/{name}` | the resource |
| List | `GET /v1/{parent}/agents` | a page of resources, see [Pagination](#pagination) |
| Create | `POST /v1/{parent}/agents` | the resource |
| Update | `PATCH /v1/{resource.name}` | the resource |
| Delete | `DELETE /v1/{name}` | empty |

## Field names

[AIP-140][aip-140]. Field names are `lower_snake_case` in protobuf and
lowerCamelCase in JSON, which is how protobuf [maps field names to
JSON][proto-json]. An OpenAPI document describes the JSON, so it uses
lowerCamelCase: `pageToken`, `nextPageToken`, `createTime`.

## Pagination

[AIP-158][aip-158]. Every method that returns a collection is paginated from
its first version. Adding pagination later breaks clients.

The request has `pageSize` and `pageToken`, both optional. The response has
the list and `nextPageToken`.

- When `pageSize` is missing or `0`, the API picks a default. It does not
  return an error.
- When `pageSize` is larger than the maximum, the API returns at most the
  maximum. It does not return an error.
- When `pageSize` is negative, the API returns `INVALID_ARGUMENT`.
- `nextPageToken` is the only signal for the end:

  > If the end of the collection has been reached, the `next_page_token`
  > field **must** be empty. This is the _only_ way to communicate
  > "end-of-collection" to users.

- A page can hold fewer items than `pageSize` and still have a next page.
  Clients page until `nextPageToken` is empty.
- Page tokens are opaque. Clients do not parse them, and a token does not
  grant access to anything.

**Don't do this**:

```json
{
  "agents": [],
  "truncated": true
}
```

A `truncated`, `hasMore` or `isLast` flag tells the client that data is
missing without giving it a way to get the rest.

**Do this**:

```json
{
  "agents": [],
  "nextPageToken": "CiAKGjBpNDd2Nmp2Zml2cXRwYjBpOXA"
}
```

## Errors

[AIP-193][aip-193]. An error response is a `google.rpc.Status`. In JSON:

```json
{
  "error": {
    "code": 404,
    "message": "Agent \"support-bot\" was not found.",
    "status": "NOT_FOUND",
    "details": [
      {
        "@type": "type.googleapis.com/google.rpc.ErrorInfo",
        "reason": "AGENT_NOT_FOUND",
        "domain": "agents.giantswarm.io",
        "metadata": {
          "agent": "support-bot"
        }
      }
    ]
  }
}
```

- `status` is one of the canonical codes of `google.rpc.Code`. The HTTP status
  follows from it.
- Every error has an `ErrorInfo` in `details`. Clients match on its `reason`
  and `domain`, never on `message`. The same pair always means the same error.
- `reason` is in `UPPER_SNAKE_CASE`, at most 63 characters.
- Permission is checked before existence. A caller who may not see a resource
  gets `PERMISSION_DENIED`, whether or not the resource exists.

[aip]: https://google.aip.dev/
[aip-122]: https://google.aip.dev/122
[aip-131]: https://google.aip.dev/131
[aip-135]: https://google.aip.dev/135
[aip-140]: https://google.aip.dev/140
[aip-158]: https://google.aip.dev/158
[aip-193]: https://google.aip.dev/193
[proto-json]: https://protobuf.dev/programming-guides/json/
