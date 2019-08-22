# Context

We use `context.Context` ([godoc][context]) as a first argument in our APIs.

This allows us to:

- Use `micrologger.Logger.LogCtx`.
- Make sure we pass the `context.Context` down the stack if we happen to
  distribute information in it.
- Allows us to handle cancellation and deadlines without changing the API.

## Guidelines:

- Don't pass `context.Context` to `New*` functions (except `NewContext` functions).
- Don't pass `context.Context` to getter and setter functions.
- Don't use `context.TODO()` in tests. It is not TODO. We are not going to
  change that.
- Otherwise always make `ctx context.Context` a first argument to the exported
  function or method.

## General pattern:

```go
// Unexported type definition ensures that only these functions can
// add or get the value from the context.
type key string
// Key instance value should be unique and otherwise doesn't matter
var customKey key = "custom"

type CustomType struct {
	Value int
}

// Encapsulate adding the value to the context
func NewContext(ctx context.Context, v *CustomType) context.Context {
	return context.WithValue(ctx, customKey, v)
}

// Encapsulate extracting the value from the context (if it exists)
func FromContext(ctx context.Context) (*CustomType, bool) {
  // Use type assertion to convert back to the value
  v, ok := ctx.Value(customKey).(*CustomType)
  return v, ok
}
```

[context]: https://golang.org/pkg/context/
