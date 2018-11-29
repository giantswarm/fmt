# Context

We use `context.Context` ([godoc][context]) as a first argument in our APIs.
This allows us to:

- Use `micrologger.Logger.LogCtx`.
- Make sure we pass the `context.Context` down the stack if we happen to
  distribute information in it.
- Allows us to handle cancellation and deadlines without changing the API.

Guidelines:

- Don't pass `context.Context` to `New*` functions.
- Don't pass `context.Context` to getter and setter functions.
- Don't use `context.TODO()` in tests. It is not TODO. We are not going to
  change that.
- Otherwise always make `ctx context.Context` a first argument to the exported
  function or method.

[context]: https://golang.org/pkg/context/
