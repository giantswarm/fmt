# Helm Checksum Annotations

## Overview

- We add checksum annotations to our deployments and daemonsets to trigger
fresh pods when their configmaps or secrets have changed.
- This reduces how often we need to roll pods which can be disruptive.
- Changes to other resources like PSP and RBAC are applied without needing to
roll pods.
- If the app has auto reload functionality like Nginx Ingress Controller these
annotations are not needed.

## Annotations

- The secret annotation is only needed if the chart has a templated secret.

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        app.giantswarm.io/config-checksum: {{ include (print .Template.BasePath "/configmap.yaml") . | sha256sum | quote }}
        app.giantswarm.io/secret-checksum: {{ include (print .Template.BasePath "/secret.yaml") . | sha256sum | quote }}
```

- Our approach is adapted from the [Helm docs](https://helm.sh/docs/howto/charts_tips_and_tricks/#automatically-roll-deployments).

## Previous Approach

- We used to use the Helm release revision number or for Helm 2 the release time.
- This triggers the rolling of pods more often which can be disruptive.

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        releaseRevision: {{ .Release.Revision | quote }}
```
