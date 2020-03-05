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

- `app` - value should contain the name of the application. Should be applied
  to every Kubernetes resource associated with an application. This together with `version`
  label should also be used as replicas selector. E.g. `app=kvm-operator,version=1.0.0`. Exceptions can
  be made to accomodate for adherence to existing selectors upstream.
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
  operator as defined in `project.go` and matching `appVersion` in
  `Chart.yaml`, e.g. `kvm-operator.giantswarm.io/version=1.0.0`. It is used by
  the given operator to recognize which object it should reconcile (i.e. to
  only reconcile objects matching its own version). When set on Node objects it
  is used to set that information in the status with the statusresource. This
  is different from release version and can be the same in multiple releases.
  Its value may be equal to that of `app.kubernetes.io/version` but it has a
  different purpose and since there could be multiple operators reconciling one
  object there could be multiple per-operator labels on one object.
- `helm.sh/chart` - value should contain a chart name and version, see
  [Helm chart labels best practices][helm-labels]
- `app.kubernetes.io/name` - the name of the application; should be applied to
  every Kubernetes resource associated with that application, i.e. all objects
  in a chart that installs it.
- `app.kubernetes.io/instance` - value should be set to `{{ .Release.Name }}`
  and is meant for differentiating between instances of the same application.
  This together with `app.kubernetes.io/name` should be used as replicas
  selector, e.g.
  `app.kubernetes.io/name=kvm-operator,app.kubernetes.io/instance=kvm-operator-1.0.0`.
- `app.kubernetes.io/version` and other common labels (see
  [here][helm-labels] and [here][k8s-common-labels]) - are informational,
  applied to all objects in a chart that installs an app or operator and so
  they indicate the source and instance of that app/operator; value of
  `.../version` should be the app/operator version as defined in `project.go`
  and matching `appVersion` in `Chart.yaml`, e.g.
  `app.kubernetes.io/version=1.0.0`.
- `app.giantswarm.io/branch` - (informational) branch from which the instance
  of the app/operator that this object is part of was built from.
- `app.giantswarm.io/commit` - (informational) ID (git SHA) of the commit from
  which the instance of the app/operator that this object is part of was built
  from.
- `giantswarm.io/provider` - value should be the installation's provider, e.g.
  `kvm`, `aws`, or `azure`.
- `version` - value should contain the version of the application.  Should be applied
  to every Kubernetes resource associated with an application. This together with `app`
  label should also be used as replicas selector. E.g. `app=kvm-operator,version=1.0.0`. Exceptions can
  be made to accomodate for adherence to existing selectors upstream.

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
