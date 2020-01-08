# How to write a changelog message for a release

## Disclaimer

The intention of this document is to help engineers to write useful changelog messages which will be part of a [release](https://github.com/giantswarm/giantswarm/wiki/Releases)
delivered to customers. Project changelogs are out of the scope here. The main goal is to use a same style across the team and
highlight the gist of the change at customer level.

## Scope

Changelogs live inside the code repositories of our components. They have a common structure defined in [versionbundle](https://godoc.org/github.com/giantswarm/versionbundle#Bundle). There are a component name, a description and a kind
property for every changelog. As an example, they look like this:

```golang
Changelogs: []versionbundle.Changelog{
      {
        Component:   "cloudconfig",
        Description: "Remove set-ownership-etcd-data-dir.service.",
        Kind:        versionbundle.KindChanged,
      },
            ...
      {
        Component:   "containerlinux",
        Description: "Update to version 1632.3.0.",
        Kind:        versionbundle.KindFixed,
        URLs: []string{
          "https://coreos.com/releases/#1632.3.0",
          "https://github.com/giantswarm/some-operator/pull/123",
        },
      },
    },
```

## Rules

### Owned components

As a result of building features and fixing bugs we change our components. Here there are some patterns about how to describe the changes:

- A changelog message describes a single change (bug fix or feature). Don't try to define more than one fix or feature per changelog message.
- Don't use the component name as the phrase subject. It is unnecessary, it heads the line already automatically. It does not add value: `aws-operator: Fix AWS operator for ELB creation`.
- Avoid writing dates or times in the message. It is already part of the release.
- For readability, group messages of one component together.
- Avoid redundant or unenriching details.
- Use lower case for component names.
- Not only say what is done, but why and how it affects the customer (In case it is clear, you can always ask SEs).
- Include a link to the pull request introducing your change. This makes it easier to find what actually changed.

### Third party components

When we only update a component like docker or container linux we don't need to write which improvements, bug fixes or features
it carries on. It is enough to specify if it is an addition, update or removal. In case of update mention which version holds. Always
add a link in the `URLs` section to the third party changelog.

## Format

Follow this common structure to make the messages and the entire release have a consistent style

`Imperative form + subject + reason`

Action list inspired by [keepachangelog.com](http://keepachangelog.com/en/1.0.0/)
`Added` `Changed` `Removed` `Fixed`

The kinds `Security` and `Deprecated` has been discarded as they are not really descriptive or does not aligned with the format.
Alternatively, you can define a security addition or modification using `Added`, `Fixed` or `Changed` providing more coherence customer-facing.

## Examples

Let's point out some Dos and Don'ts.

### 1. Why you should not repeat the component name in the message

#### Bad

There is a redundant component name, bad verb tense and no focus in what improves for the client, final dot)

```golang
{
    Component:   "aws-operator",
    Description: "Fix aws operator ELB tags",
    Kind:        versionbundle.KindChanged,
}
```

Output in our user interfaces:

```
aws-operator: Fix aws operator ELB tags
```

#### Good

```golang
{
    Component:   "aws-operator",
    Description: "Fix limitation getting ELB tags that affected cluster creation.",
    Kind:        versionbundle.KindFixed,
    URLs: []string{
      "https://github.com/giantswarm/aws-operator/pull/123",
    },
}
```

Output in our user interfaces:

```
aws-operator: Fixed limitation getting ELB tags that affected cluster creation.
```

### 2. When a third party library is updated

#### Bad

Wrong verb tense, chossing wrong verb, provide link to release notes if it is possible)

```golang
{
    Component:   "Docker",
    Description: "Bump version to 1.12.",
    Kind:        versionbundle.KindChanged,
}
```

Output:

```
docker: Bump version to 1.12
```

#### Good

```golang
{
    Component:   "docker",
    Description: "Update to 1.12.",
    Kind:        versionbundle.KindChanged,
    URLs: []string{
      "https://docs.docker.com/enterprise/17.03/release-notes/",
    },
}
```

Output in our user interfaces (the link is currently ommitted in happa
but shown in the `releases` repo):

```
docker: Update to 1.12.
```

