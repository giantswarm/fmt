# Log Messages

We log with structured loggers: [`log/slog`][slog] in services and CLIs, and
the [`logr`][logr] logger that controller-runtime puts in the context in
controllers (`log.FromContext(ctx)`). `github.com/giantswarm/micrologger` is
deprecated; new code does not use it.

- [Messages and attributes](#messages-and-attributes)
- [Levels](#levels)
- [Errors](#errors)
- [Reconciliation](#reconciliation)

## Messages and attributes

The message says what happened, in lowercase, without formatting values into
it. Values go into attributes, so they can be searched and filtered:

```go
logger.Info("created app", "name", app.Name, "namespace", app.Namespace)
```

**Don't do this**:

```go
logger.Info(fmt.Sprintf("created App CR %q in namespace %q", app.Name, app.Namespace))
```

- Attribute keys are lowerCamelCase and mean the same thing everywhere in the
  repository: `namespace` is always a namespace, never sometimes a cluster.
  Where an [OpenTelemetry semantic convention][semconv] names the value, use
  that key.
- Add attributes that every message of a request or reconciliation needs once,
  with `logger.With(...)`, not on every call.
- Never log secrets, tokens or full request bodies.

## Levels

- **Error**: something failed and nobody else will report it. See
  [Errors](#errors).
- **Info**: a change the program made, or a decision a person reading the logs
  needs: created, deleted, skipped because.
- **Debug**: the steps in between: what it found, what it is about to do.

A controller logs a change it made, not every reconciliation that changed
nothing.

## Errors

An error is either returned or logged, not both. The code that decides not to
return it logs it once, with the error as an attribute:

```go
for _, c := range collectors {
	if err := c.Collect(ch); err != nil {
		logger.Error("collect metrics", "collector", c.Name(), "error", err)
	}
}
```

With `logr`, the error is the first argument:
`logger.Error(err, "collect metrics", "collector", c.Name())`.

## Reconciliation

Logs of a reconciliation are read by SREs during incidents, so they say what the
controller found and what it did, in plain words:

```go
logger.Info("deleting app", "reason", "cluster is being deleted")
// ...
logger.Info("deleted app")
```

When a reconciliation waits for something, log what it waits for and requeue,
rather than returning an error:

```go
logger.Info("waiting for worker node pool", "nodePool", name)
return ctrl.Result{RequeueAfter: 30 * time.Second}, nil
```

[logr]: https://pkg.go.dev/github.com/go-logr/logr
[semconv]: https://opentelemetry.io/docs/specs/semconv/
[slog]: https://pkg.go.dev/log/slog
