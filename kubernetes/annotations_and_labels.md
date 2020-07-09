# Kubernetes Annotations and Labels

This page defines common annotations and labels we set in Kubernetes objects.

## Common Annotations

- `giantswarm.io/docs` - value should be an URL to a common documentation
  location. For now this should be in our `giantswarm/giantswarm` repository in
  which we crosslink all pages to create a reasonable documentation of all kinds
  of Kubernetes resources, their functionality and relationships.
- `machine-deployment.giantswarm.io/subnet` - value contains a subnet CIDR that
  has been allocated for the given node pool / MachineDeployment object that
  has this annotation. E.g.
  `machine-deployment.giantswarm.io/subnet=10.102.31.0/24`.

## Common Labels

- `app` - should be set to the same value as `app.kubernetes.io/name`. It's
  deprecated and only kept to avoid breaking existing workflows.
- `giantswarm.io/certificate` - value should contain certificate name as
  defined in github.com/giantswarm/certs repo. This is used in certificate
  Secrets and CertConfigs.
- `giantswarm.io/cluster` - value should contain tenant cluster ID which this
  object is part of. E.g. `giantswarm.io/cluster=eggs2`.
- `giantswarm.io/machine-deployment` - value should contain tenant cluster node
  pool ID (i.e. the machine deployment ID) which this object is part of. E.g.
  `giantswarm.io/machine-deployment=al9qy`.
- `giantswarm.io/managed-by` - value should contain repository name of the
  operator managing the object. E.g. `giantswarm.io/managed-by=kvm-operator`.
- `giantswarm.io/organization` - value should contain tenant cluster's
  organization ID as displayed in the front-end
  `giantswarm.io/organization=dev`.
- `giantswarm.io/randomkey` - value should contain randomkey name as defined in
  github.com/giantswarm/randomkeys repo. This is used in encryption Secrets.
- `giantswarm.io/service-type` - value should be either `system` for core system
  services (i.e. K8s components) or `managed` for Giant Swarm managed services
  (i.e. networking, DNS, monitoring, ingress controller, etc.). Latter would be
  managed by `chart-operator`.
- `release.giantswarm.io/version` - value should be Giant Swarm release
  version, e.g. `release.giantswarm.io/version=2.3.0`.
- `OPERATOR.giantswarm.io/version` - value should be the version of the
  given operator reconciling the object, as defined in its `project.go` and
  `appVersion` in `Chart.yaml`, e.g.
  `kvm-operator.giantswarm.io/version=1.0.0`. It is used by the given operator
  to recognize which object it should reconcile (i.e. to only reconcile objects
  matching its own version, which is exposed as `app.kubernetes.io/version` on
  the operator's own component resources like `Deployment`). When set on Node
  objects it is used to set that information in the status with the
  statusresource. This is different from release version and can be the same in
  multiple releases. Since there could be multiple operators reconciling one
  object there could be multiple per-operator labels on one object.
- `helm.sh/chart` - value should contain a chart name and version, see
  [Helm chart labels best practices][helm-labels]
- `app.kubernetes.io/name` - the name of the application; should be applied to
  every Kubernetes resource associated with that application, i.e. all objects
  in a chart that installs it. E.g. `app.kubernetes.io/name=kvm-operator`.
- `app.kubernetes.io/instance` - value should be set to `{{ .Release.Name }}`
  and is meant for differentiating between instances of the same application.
  This together with `app.kubernetes.io/name` should be used as replicas
  selector, e.g.
  `app.kubernetes.io/name=kvm-operator,app.kubernetes.io/instance=kvm-operator-1.0.0`.
- informational labels, applied to all objects in a chart that installs an app
  or operator and so they indicate its source and instance:
  - `app.kubernetes.io/version` - value should be the app/operator version as
    defined in `project.go` and matching `appVersion` in `Chart.yaml`, e.g.
    `app.kubernetes.io/version=1.0.0`. In case of our operators there should be
    only a single instance with a given version running on a cluster as this
    also indicates this operator reconciles objects labeled with
    `OPERATOR.giantswarm.io/version` with the same value, and there should be
    only one **version** of an operator reconciling a single object. This is
    enforced by GateKeeper on our clusters.
  - `app.giantswarm.io/branch` - branch from which the instance of the
    app/operator that this object is part of was built from.
  - `app.giantswarm.io/commit` - ID (git SHA) of the commit from which the
    instance of the app/operator that this object is part of was built from.
- `giantswarm.io/provider` - value should be the installation's provider, e.g.
  `kvm`, `aws`, or `azure`.

Also see [Helm chart labels best practices][helm-labels] and
[common labels recommended in Kubernetes docs][k8s-common-labels] for a set of
common labels that can be set on all objects to enable visualisation and
querying by shared tooling.


[helm-labels]: https://helm.sh/docs/topics/chart_best_practices/labels/
[k8s-common-labels]: https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/

### Labels Set In Custom Resources

#### Control Plane

- `giantswarm.io/managed-by`
- `OPERATOR.giantswarm.io/version`
- `release.giantswarm.io/version`

#### Tenant Cluster

- `giantswarm.io/managed-by`
- `OPERATOR.giantswarm.io/version`
- `release.giantswarm.io/version`


### Annotations and Labels Set In Cluster API Custom Resource Objects

#### App Catalog

#### Annotation

- `giantswarm.io/monitoring` - `true|false', indicating monitoring feature on/off on the service.
- `giantswarm.io/monitoring-path` - value should indicate monitoring URL path.
- `giantswarm.io/monitoring-port` - value should indicate monitoring port on the specific path.

##### Labels

- `application.giantswarm.io/catalog-type`
  Describes the type of catalog that this AppCatalog CR represents.
  This label helps our UI determine how to show this app catalog.

  - `internal` - Will not show up in our UIs at all.
  - `managed` - Will gain a 'managed' banner, helping it stand out from other catalogs.
  - `incubator` - Will show some expectation management message before installing an app from this catalog, that it is still a work in progress, but expected to at least install and somewhat work.
  - `community` - Will show a more strongly worded expectation management message, indicating that  apps from this catalog will most likely not work without some adjustments.

#### Cluster

##### Annotations

##### Labels

#### MachineDeployment

##### Annotations

- `machine-deployment.giantswarm.io/subnet`

##### Labels

- `giantswarm.io/machine-deployment`

### Labels Set In Nodes

#### Control Plane

#### Tenant Cluster

- `release.giantswarm.io/version` - Should be applied to all Kubernetes nodes,
  regardless of their role.
- `OPERATOR.giantswarm.io/version` - Should be applied to all Kubernetes nodes,
  regardless of their role. This should be set by the operator itself and never
  change during the node lifetime.
- `giantswarm.io/provider` - Should be applied to all Kubernetes nodes,
  regardless of their role.

### Labels Set In Objects Created With Managed Services

#### Control Plane

#### Tenant Cluster

- `app`
- `giantswarm.io/service-type`

### Labels Set In Objects Created With Operators

#### Control Plane

- `giantswarm.io/cluster`
- `giantswarm.io/managed-by`
- `giantswarm.io/organization`

#### Tenant Cluster

- `giantswarm.io/managed-by`

### Labels Set In Secrets

#### Control Plane

- `giantswarm.io/cluster`
- `giantswarm.io/certificate` - Only when `giantswarm.io/randomkey` is not set.
- `giantswarm.io/managed-by`
- `giantswarm.io/organization`
- `giantswarm.io/randomkey` - Only when `giantswarm.io/certificate` is not set.

#### Tenant Cluster

### Labels Set On Objects Installed By Charts

Labels related to a higher-level virtual concept of an _app_, i.e. a bunch of
components usually installed by one or more Helm charts.

- `app`
- `app.kubernetes.io/name`
- `app.kubernetes.io/instance`
- `app.giantswarm.io/branch`
- `app.giantswarm.io/commit`
- `app.kubernetes.io/managed-by`
- `app.kubernetes.io/version`
- `helm.sh/chart`

To set those labels without having to repeat their definition in multiple
places we use a template helper based on that in [chart template][helm-def-tpl]
from upstream Helm. Template helpers are any file that begin with an underscore
and should be placed inside the chart's `templates/` subdirectory and they
define a couple of variables that can then be re-used throughout other
templates in the chart.

Contents of `_helpers.tpl`:

```
{{/* vim: set filetype=mustache: */}}
{{/*
Expand the name of the chart.
*/}}
{{- define "name" -}}
{{- .Chart.Name | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{/*
Create chart name and version as used by the chart label.
*/}}
{{- define "chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{/*
Common labels
*/}}
{{- define "labels.common" -}}
app: {{ include "name" . | quote }}
{{ include "labels.selector" . }}
app.giantswarm.io/branch: {{ .Values.project.branch | quote }}
app.giantswarm.io/commit: {{ .Values.project.commit | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service | quote }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
helm.sh/chart: {{ include "chart" . | quote }}
{{- end -}}

{{/*
Selector labels
*/}}
{{- define "labels.selector" -}}
app.kubernetes.io/name: {{ include "name" . | quote }}
app.kubernetes.io/instance: {{ .Release.Name | quote }}
{{- end -}}
```

Contents of `_resource.tpl`:

```
{{/* vim: set filetype=mustache: */}}
{{/*
Create a name stem for resource names

When pods for deployments are created they have an additional 16 character
suffix appended, e.g. "-957c9d6ff-pkzgw". Given that Kubernetes allows 63
characters for resource names, the stem is truncated to 47 characters to leave
room for such suffix.
*/}}
{{- define "resource.default.name" -}}
{{- .Release.Name | replace "." "-" | trunc 47 | trimSuffix "-" -}}
{{- end -}}

{{- define "resource.default.namespace" -}}
giantswarm
{{- end -}}

{{- define "resource.psp.name" -}}
{{- include "resource.default.name" . -}}-psp
{{- end -}}

{{- define "resource.pullSecret.name" -}}
{{- include "resource.default.name" . -}}-pull-secret
{{- end -}}
```

You'll also need to add the following snippet to the chart's `values.yaml`
(the placeholders are automatically replaced by `architect` during build):

``` yaml
project:
  branch: "[[ .Branch ]]"
  commit: "[[ .SHA ]]"
```

This defines a couple of template variables based on chart name/version
normalised to comply with k8s resource name [constraints][k8s-identifiers], and
two snippets with labels:

- `name` - name of the chart, trimmed to 63 characters
- `chart` - normalised name + version of the chart, i.e. trimmed to 63
  characters and with `+` signs replaced with `-`
- `labels.common` - defines all the labels described above, including the
  selector labels;
  usage: `{{- include "labels.common" . | nindent INDENT }}`
  (set `INDENT` to required number of spaces)
- `labels.selector` - defines labels to be used in selectors;
  usage: `{{- include "labels.selector" . | nindent INDENT }}`
  (set `INDENT` to required number of spaces)
- `resource.default.name` - default name for k8s resources created by the chart
  usage: `{{ include "resource.default.name"  . }}`
- `resource.default.namespace` - namespace for all resources in the chart
- `resource.psp.name` - default name for _PodSecurityPolicy_ resources in the
  chart
- `resource.pullSecret.name` - default name for pull secrets

<details>
<summary>EXAMPLE</summary>
<p>
For example, this snippet from <code>templates/deployment.yaml</code> in <em>aws-operator</em>:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "resource.default.name"  . }}
  namespace: {{ include "resource.default.namespace"  . }}
  labels:
    {{- include "labels.common" . | nindent 4 }}
spec:
  replicas: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      {{- include "labels.selector" . | nindent 6 }}
  strategy:
    type: RollingUpdate
  template:
    metadata:
      annotations:
        releasetime: {{ $.Release.Time }}
      labels:
        {{- include "labels.selector" . | nindent 8 }}
    spec:
...
```

produces the following when rendered:

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aws-operator-8-2-1-dcff541
  namespace: giantswarm
  labels:
    app: "aws-operator"
    app.kubernetes.io/name: "aws-operator"
    app.kubernetes.io/instance: "aws-operator-8.2.1-dcff541"
    app.giantswarm.io/branch: "voo-ensure-labels"
    app.giantswarm.io/commit: "5b8a2f2e457e7a0f95084b32a43ce88959fd2552"
    app.kubernetes.io/managed-by: "Helm"
    app.kubernetes.io/version: "8.2.2-dev"
    helm.sh/chart: "aws-operator-8.2.1-5b8a2f2e457e7a0f95084b32a43ce88959fd2552"
spec:
  replicas: 1
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app.kubernetes.io/name: "aws-operator"
      app.kubernetes.io/instance: "aws-operator-8.2.1-dcff541"
  strategy:
    type: Recreate
  template:
    metadata:
      annotations:
        releasetime: 
      labels:
        app.kubernetes.io/name: "aws-operator"
        app.kubernetes.io/instance: "aws-operator-8.2.1-dcff541"
    spec:
...
```
</p>
</details>


[helm-def-tpl]: https://github.com/helm/helm/blob/ec1d1a3d3eb672232f896f9d3b3d0797e4f519e3/pkg/chartutil/create.go#L338
[k8s-identifiers]: https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/identifiers.md

## Finalizers

- Operatorkit sets a finalizer for objects that are watched by the framework
  named `operatorkit.giantswarm.io/NAME`.
- `NAME` is configured by the operator and should be the operator name. E.g.
  `kvm-operator`.
- If one operator implements multiple frameworks, then `NAME` should include
  the framework name. E.g `aws-operator-cluster` and
  `aws-operator-drainer`.

## Naming

- Names should consist of lowercase letters, numbers, dashes and at most one
  slash (this is enforced by the Kubernetes API).
- Company specific annotations and labels should be prefixed with
  `giantswarm.io/`. E.g `giantswarm.io/cluster-id`.
- Operator specific annotations and labels should be prefixed with
  `OPERATOR.giantswarm.io/`. E.g. `kvm-operator.giantswarm.io/service`.
