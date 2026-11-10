# Names

A name tells the reader which concept they are looking at. When our code calls
a thing by the name its specification uses, the reader can look it up. When we
invent a name, or use a known name for something else, the reader has to learn
our private meaning first, and will get it wrong.

This applies to everything that carries a name: identifiers, types, API
fields, CRD fields, flags, metrics, log keys, documentation and pull requests.

- [Find the term before you name](#find-the-term-before-you-name)
- [One meaning per word](#one-meaning-per-word)
- [Names on the wire](#names-on-the-wire)
- [Plain words](#plain-words)
- [Project glossary](#project-glossary)

## Find the term before you name

We look for an existing term in this order and use the first one we find:

1. The specification the code implements: the RFC, the MCP schema, the AIP,
   the Kubernetes API conventions.
2. The upstream project the code builds on.
3. The repository itself: its [glossary](#project-glossary), then its existing
   code and documentation.

When none of them names the concept, we choose a name and say in the doc
comment that it is ours. We do not present an invented word as a standard one.

**Don't do this**: call the party a token was issued for the "owner" in OAuth
code. RFC 8693 calls it the subject.

## One meaning per word

Inside one repository, a word means one thing, and a thing has one word.

- We do not reuse a term of the specification for something else. In code that
  handles OpenID Connect, a claim is a claim of a token. A lock that a worker
  takes on a job is not a "claim" there.
- We do not use two words for the same thing. If the API says `agent`, the
  code, the logs and the documentation do not also say "bot" or "assistant".

## Names on the wire

A name that leaves the process is expensive to change: a JSON field, a CRD
field, a flag, a metric, a label. Renaming it later needs a migration and
breaks clients. We check these names against the specification before the
first release that contains them.

## Plain words

Documentation, comments and pull requests describe what the code does in
words any engineer knows. We do not use the vocabulary of a design method to
describe our code: no "aggregate", "bounded context", "port", "adapter" or
"anti-corruption layer". Say what the thing is: a package, a client, an
interface, a table.

## Project glossary

A repository that defines its own concepts keeps a glossary in
`docs/glossary.md`. Examples of such concepts are API resources, custom
resources, and the things its documentation has to explain. Forks and
repositories without concepts of their own do not need one.

An entry has the term, one sentence that says what it is, and the words we do
not use for it:

```markdown
## Agent

A configured assistant that a person can start sessions with. An agent has a
model, instructions and tools.

Not: bot, assistant, persona.
```

- The glossary holds terms the repository defines, or uses in a narrower way
  than usual. A term of a specification is a link to the section that defines
  it, not a new definition.
- A term that several repositories share is defined in one of them. The others
  link to it.
- A pull request that introduces a new term adds it to the glossary.
