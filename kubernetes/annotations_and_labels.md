# Kubernetes Annotations and Labels

## Common Labels

- Deployments should have `app` label. This label should be a selector for
  number of replicas.
- All Kubernetes object managed by an operator should be labeled with
  `giantswarm.io/managed-by` with value set to the operator repository name.

## Naming

- Operator specific labels should be prefixed with
  `OPERATOR_REPO.giantswarm.io/`. E.g. `kvm-operator.giantswarm.io/cluster-id`.
- Operator specific annotations should be prefixed with
  `OPERATOR_REPO.giantswarm.io/`. E.g. `endpoint-operator.giantswarm.io/service`.
