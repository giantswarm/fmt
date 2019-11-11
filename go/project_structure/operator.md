# Operator

This page describes common code structure for our operators.

The example below is taken from [aws-operator] on 2019/11/01. Most of the files
and directories are elided for simplification. Note that not all packages are
described as this page should be extended over time.

```
./service/
├── collector/
├── controller/
│   ├── controllercontext/
│   ├── internal/
│   │   ├── adapter/
│   │   ├── changedetection/
│   │   ├── cloudconfig/
│   │   ├── cloudformation/
│   │   ├── ebs/
│   │   ├── encrypter/
│   │   ├── templates/
│   │   └── unittest/
│   ├── key/
│   └── resource/
│   │   ├── accountid/
│   │   ├── ...
│   │   └── tcnpoutputs/
│   ├── cluster.go
│   └── machine_deployment.go
├── internal/
│   ├── accountid/
│   ├── credential/
│   └── locker/
└── service.go
```

- `service/collector` package contains code for Prometheus collectors.
- `service/controller/controllercontext` package contains controller context
  code used for communication between controller resources. This information
  has a scope of a single reconciliation loop. I.e. each loops starts with an
  empty context.
- `service/controller/internal` package contains sub-packages used by controller
  resources.
- `service/controller/key` package contains common values extracting logic and
  constants shared between controller resources.
- `service/controller/resource` sub-packages contain code for a single
  operatorkit resource each. The name of each sub-package is also the name for
  the resource.
- `service/controller/cluster.go` is the source code wiring "cluster"
  controller reconciling Cluster CRs.
- `service/controller/machine_deployment.go` is the source code wiring
  "machinedeployment" reconciling MachineDeployment CRs.
  controller.
- `service/internal` package contains sub-packages used by both `service/controller`
  and `service/collector` or dependencies wired into multiple controllers.
- `service/service.go` is the source code wiring all controllers and
  collectors.

[aws-operator]: https://github.com/giantswarm/aws-operator
