# Kubernetes Annotations and Labels

## Common Labels

- `app` - value should contain the name of the application. Should be applied
  to Deployments and Pods. This should be also used as replicas selector.
- `giantswarm.io/cluster-id` - value should contain guest cluster ID which this
  object is part of. E.g. `giantswarm.io/cluster-id=eggs2`.
- `giantswarm.io/customer-id` - value should contain guest cluster's customer
  ID. `giantswarm.io/customer-id=track-hunter`.
- `giantswarm.io/managed-by` - value should contain repository name of the
  operator managing the object. E.g. `giantswarm.io/managed-by=kvm-operator`.

## Naming

- Names should consist of lowercase letters, numbers, dashes and at most one
  slash (this is enforced by the Kubernetes API).
- Company specific annotations and labels should be prefixed with
  `giantswarm.io/`. E.g `giantswarm.io/cluster-id`.
- Operator specific annotations and labels should be prefixed with
  `OPERATOR_REPO.giantswarm.io/`. E.g.
  `endpoint-operator.giantswarm.io/service`.
