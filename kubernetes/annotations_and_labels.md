# Kubernetes Annotations and Labels

This page defines common annotations and labels we set in Kubernetes objects.

## Common Annotations

- `giantswarm.io/docs` - value should be an URL to a common documentation
  location. For now this should be in our `giantswarm/giantswarm` repository in
  which we crosslink all pages to create a reasonable documentation of all kinds
  of Kubernetes resources, their functionality and relationships.
- `giantswarm.io/version-bundle` - for CRs exposed to customers we will use
  this annotation instead of having it in the spec. As this is an implementation
  detail for our operators.

## Common Labels

- `app` - value should contain the name of the application. Should be applied
  to every Kubernetes resource associated with an application. This should also
  be used as replicas selector. E.g. `app=kvm-operator`. Exceptions can be made
  to accomodate for adherence to existing selectors upstream.
- `giantswarm.io/cluster` - value should contain tenant cluster ID which this
  object is part of. E.g. `giantswarm.io/cluster=eggs2`.
- `giantswarm.io/managed-by` - value should contain repository name of the
  operator managing the object. E.g. `giantswarm.io/managed-by=kvm-operator`.
- `giantswarm.io/organization` - value should contain tenant cluster's
  organization ID as displayed in the front-end
  `giantswarm.io/organization=adidas`.
- `giantswarm.io/randomkey` - value should contain randomkey name as defined in
  github.com/giantswarm/randomkeys repo. This is used in encryption Secrets.
- `giantswarm.io/service-type` - value should be either `system` for core system
  services (i.e. K8s components) or `managed` for Giant Swarm managed services
  (i.e. networking, DNS, monitoring, ingress controller, etc.). Latter would be
  managed by `chart-operator`.
- `release.giantswarm.io/version` - value should be the used Giant Swarm release
  version, e.g. `2.3.0`.
- `OPERATOR.giantswarm.io/version` - value should be the used operator version
  bundle version, e.g. `kvm-operator.giantswarm.io/version=1.0.0`.
- `giantswarm.io/provider` - value should be the installation's provider, e.g.
  `kvm`, `aws`, or `azure`.
- `giantswarm.io/certificate` - value should contain certificate name as
  defined in github.com/giantswarm/certs repo. This is used in certificate
  Secrets and CertConfigs.

### Labels Set In Tenant Cluster Nodes

- `release.giantswarm.io/version`
- `OPERATOR.giantswarm.io/version`
- `giantswarm.io/provider`

### Labels Set In Control Plane Objects

- `app`
- `giantswarm.io/cluster`
- `giantswarm.io/certificate`
- `giantswarm.io/managed-by`
- `giantswarm.io/organization`
- `giantswarm.io/randomkey`

### Labels Set In Tenant Cluster Objects

- `app`
- `giantswarm.io/service-type`

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
