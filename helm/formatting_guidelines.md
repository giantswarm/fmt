# Helm Formatting Guidelines

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
# No spaces makes it harder to see the templating call
name: {{.Release.Name}}-configmap
```

## Indendation

The indendation of our helm chart yaml files should be 2 spaces.
Moreover we do not indend yaml lists, hyphens of each list member act as indentation.
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
