# Kubernetes Annotations and Labels

## Common Labels (for host cluster services)

- `app` - value should contain the name of the application. Should be applied
  to every Kubernetes resource associated with an application. This should
  also be used as replicas selector. E.g. `app=kvm-operator`.
- `giantswarm.io/cluster` - value should contain guest cluster ID which this
  object is part of. E.g. `giantswarm.io/cluster=eggs2`.
- `giantswarm.io/managed-by` - value should contain repository name of the
  operator managing the object. E.g. `giantswarm.io/managed-by=kvm-operator`.
- `giantswarm.io/organization` - value should contain guest cluster's
  organization ID as displayed in the front-end
  `giantswarm.io/organization=track-hunter`.
- `giantswarm.io/randomkey` - value should contain type of the randomkey stored
  in secret. E.g. `giantswarm.io/randomkey=encryption` for guest API -> etcd
  encryption at rest.

## Common Labels (for guest cluster services)

- `app` - value should contain the name of the application. Should be applied
  to every Kubernetes resource associated with an application. This should 
  also be used as replicas selector. E.g. `app=nginx-ingress-controller`.
  Exceptions can be made to accomodate for adherence to existing selectors
  upstream.
- `giantswarm.io/service-type` - value should be either `system` for core system
  services (i.e. K8s components) or `managed` for Giant Swarm managed services
  (i.e. networking, DNS, monitoring, ingress controller, etc.). Latter would be
  managed by `chart-operator`.

## Finalizers

- Operatorkit sets a finalizer for custom objects that are watched by the
  framework named `operatorkit.giantswarm.io/NAME`.
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
  `OPERATOR_REPO.giantswarm.io/`. E.g.
  `endpoint-operator.giantswarm.io/service`.
