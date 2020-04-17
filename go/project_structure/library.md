# Library

This page describes common code structure for our libraries.

The example below is taken from [k8sclient] on 2020/04/16. Most of the files
and directories are elided for simplification.

```
github.com/giantswarm/k8sclient/
├── CHANGELOG.md
├── README.md
├── go.mod
├── go.sum
└── pkg
    ├── k8sclient
    │   ├── clients.go
    │   ├── error.go
    │   ├── setup.go
    │   ├── spec.go
    │   ├── spec_test.go
    │   └── types.go
    ├── k8sclienttest
    │   └── clients.go
    ├── k8scrdclient
    │   ├── crd_client.go
    │   ├── crd_client_test.go
    │   ├── error.go
    │   └── spec.go
    ├── k8srestconfig
    │   ├── error.go
    │   └── rest_config.go
    └── k8sversion
        ├── error.go
        ├── funcs.go
        ├── funcs_test.go
        └── types.go
```

Below the go.mod file and a few packages discussed.

## go.mod file

Since we use Go modules releasing a major version of the library means we also
need to change the import path of the module. The last segment of the module
import must be `/vX` for `vX.Y.Z` releases where `X >= 2`. E.g. for release
`v3.4.5` it must be `v3`. This is important for the libraries as they are
imported in external repositories (as opposed to operators for example) and
they wouldn't be Go module compatible otherwise. It's worth mentioning that
module defined with `/vX` segment is treated as a separate module from the v0
and v1 versions and they can be imported at the same time.

Let's try to illustrate that with `github.com/giantswarm/k8sclient` library
example.

For releases `v0.x.x` and `v1.x.x` in the `go.mod` file there should be a line
like:

```
module github.com/giantswarm/k8sclient
```

For releases `v2.x.x`:

```
module github.com/giantswarm/k8sclient/v2
```

For releases `v3.x.x`:

```
module github.com/giantswarm/k8sclient/v3
```

And so on.

Note that even though the `module` directive changes on major version bumps the
directory layout inside the repository does not.

## Root package

Since we use Go modules we don't have any Go files in the root of the library
repository. This is because as we bump the major version of the module we need
to use `/vN` imports. We put everything under subpackages of `/pkg` to avoid
some ugly import aliases like:

```
k8sclientv3 "github.com/giantswarm/k8sclient/v3"
```

Instead the import will look like:

```
"github.com/giantswarm/k8sclient/v3/pkg/k8sclient"
```

This comes at expense of longer imports of other packages. E.g. instead of
having import like:

```
"github.com/giantswarm/k8sclient/v3/k8scrdclient"
```

We have import like:

```
"github.com/giantswarm/k8sclient/v3/pkg/k8scrdclient"
```

Which seems to be a fair price to pay.

## k8scrdclient

```
./pkg/k8scrdclient
├── crd_client.go
├── crd_client_test.go
├── error.go
└── spec.go
```

- `crd_client.go` holds `CRDClient` type implementation and `Config` type it is
  configured with.
- `crd_client_test.go` holds tests for `CRDClient` type.
- `error.go` holds error definitions for the package.
- `spec.go` holds `Interface` type which is interface for this package (in this
  case interface which `CRDClient` type implements).

This is a typical layout for a package holding a single type providing methods
executing some business logic. In this case `*k8scrdclient.CRDClient`. We often
have corresponding test package, e.g. `k8scrdclienttest` implementing a mock
object implementing `Interface` type.

## k8sversion

```
./pkg/k8sversion
├── error.go
├── funcs.go
├── funcs_test.go
└── types.go
```

- `error.go` holds error definitions for the package.
- `funcs.go` holds functions which are not methods of any struct (in this case
  `Latest` and `Less`).
- `types.go` holds pure data structs. Those are structs which don't have
  methods executing business logic. They are used to carry data around.
  Preferably they are not mutated after creation.

This is a typical layout for a package exposing functions. In this case
`k8sversion.Latest` and `k8sversion.Less`.
