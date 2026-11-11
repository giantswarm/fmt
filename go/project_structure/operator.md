# Operator

New operators use [kubebuilder][kubebuilder] and
[controller-runtime][controller-runtime], with the project layout kubebuilder
generates. `github.com/giantswarm/operatorkit` is deprecated; existing
operatorkit operators keep their structure until they are migrated.

- One reconciler per kind, in `internal/controller/`.
- API types in `api/<version>/`, following the
  [Kubernetes API conventions][k8s-api]. See
  [Specifications](../../design/specifications.md).
- Reconcilers are idempotent. See [Idempotency](../idempotency_conventions.md).

[controller-runtime]: https://github.com/kubernetes-sigs/controller-runtime
[k8s-api]: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md
[kubebuilder]: https://book.kubebuilder.io/
