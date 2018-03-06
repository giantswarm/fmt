# How write a changelog message for a release

## Disclaimer

The intention of this document is to help engineers to write useful changelog messages which will be part of a [release](https://github.com/giantswarm/giantswarm/wiki/Releases)
delivered to customers. Project changelogs are out of the scope here. The main goal is use a same style across the team and
highlight the gist of the change at customer level. 
 
## Rules

### Owned components

As a result of building features and fixing bugs we change our components. Here there are some patterns about how describe the changes:

- A changelog message describes a single change (bug fix or feature). Don't try to define more than fix or feature by changelog message.    
- Don't use the component name as the phrase subject. It is unnecessary, it heads the line already automatically. It does not add value: `AWS operator: Fixed AWS operator for ELB creation`.
- Avoid writing dates or times in the message. It is already part of the release.
- For readability, group messages of one component together.
- Avoid redundant or unenriching details.
- Use title case for component names.
- Not only say what is done, but why and how it affects the customer.

### Third party components

When we only update a component like docker or container linux we don't need to write which improvements, bug fixes or features
it carries on. It is enough to specify if it is an addition, update or removal. In case of update mention which version holds. Also
will be great add the link to upstream changelogs or release page.

## Format

Follow this common structure to make the messages and the entire release have a consistent style

`Imperative + subject + reason`

Imperative list inspired by [keepachangelog.com](http://keepachangelog.com/en/1.0.0/)
`Added` `Changed` `Removed` `Fixed` `Security`

## Examples

Let's stand out some good created changelogs messages

```
BAD (redundant component name, bad verb tense and no focus in what improves for the client)
aws-operator: Fix aws operator ELB tags.
GOOD
AWS Operator: Fixed limitation getting ELB tags that affected cluster creation.
```

For single third party updates

```
BAD
docker: Bump version to 1.12
GOOD
Docker: Updated to 1.12.
```