# Kubernetes Annotations and Labels

## Common Labels

- `app` - value should contain the name of the application. Should be applied
  to every Kubernetes resource associated with an application. This should be
  also used as replicas selector. E.g. `app=kvm-operator`.
- `giantswarm.io/cluster` - value should contain guest cluster ID which this
  object is part of. E.g. `giantswarm.io/cluster=eggs2`.
  operator managing the object. E.g. `giantswarm.io/managed-by=kvm-operator`.
- `giantswarm.io/managed-by` - value should contain repository name of the
- `giantswarm.io/organization` - value should contain guest cluster's
organization ID as displayed in the front-end
`giantswarm.io/organization=track-hunter`.
- `giantswarm.io/randomkey` - value should contain type of the randomkey stored
  in secret. E.g. `giantswarm.io/randomkey=encryption` for guest API -> etcd
  encryption at rest.

## Finalizers

- Operator specific finalizers for custom objects shoud be named
  `OPERATOR_REPO.giantswarm.io/custom-object-cleanup`. E.g.
  `ingress-operator.giantswarm.io/custom-object-cleanup`.

## Naming

- Names should consist of lowercase letters, numbers, dashes and at most one
  slash (this is enforced by the Kubernetes API).
- Company specific annotations and labels should be prefixed with
  `giantswarm.io/`. E.g `giantswarm.io/cluster-id`.
- Operator specific annotations and labels should be prefixed with
  `OPERATOR_REPO.giantswarm.io/`. E.g.
  `endpoint-operator.giantswarm.io/service`.
