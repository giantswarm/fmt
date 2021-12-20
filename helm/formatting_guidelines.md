# Helm Chart Formatting Guidelines

## Templating

When using helm templating calls we include double spaces inside the double brackets
like this:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Hello World"
```

In comparison to this:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  # No spaces makes it harder to see the templating call
  name: {{.Release.Name}}-configmap
data:
  myvalue: "Hello World"
```

When helm templating calls access values with hyphens in the name you need to
use the `index` function. Otherwise the error `bad character U+002D '-'` will
be returned by helm when installing the chart.

```yaml
proxy-buffers-size: "{{ index .Values.configmap "proxy-buffers-size" }}"
```

## Indentation

The indentation of our helm chart yaml files should be 2 spaces.
Moreover we do not indent yaml lists, hyphens of the list members act as indentation.

Example:
```yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: {{ .Values.controller.name }}
  namespace: {{ .Values.namespace }}
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - endpoints
  - nodes
  - pods
  - secrets
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
```

## Namespace

Namespaces are optional for resources in helm charts in general. This allows
`helm template | kubectl apply -f -n <namespace> -` to install all resources into
the specified namespace as expected. This approach has problems when installing cluster-scoped
resources such as `ClusterRole`s as these need to refer to a `ServiceAccount` in a specific
namespace. Additionally, some helm charts install resources only in well-known namespaces such
as `kube-system`.

To ensure consistency and avoid the above problems, it is preferable to specify
`{{ .Release.Namespace }}` as the value for `.metadata.namespace` and other namespace fields of
resources in a helm chart except when resources need to go into a specific hardcoded namespace.
