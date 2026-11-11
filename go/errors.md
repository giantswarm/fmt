# Errors

We use the standard library's [`errors`][errors] package and `fmt.Errorf`.
Existing code that uses `github.com/giantswarm/microerror` keeps it until it is
changed for another reason; new code does not use it.

- [Wrapping](#wrapping)
- [Errors callers match](#errors-callers-match)
- [Package boundaries](#package-boundaries)
- [Zero values](#zero-values)

## Wrapping

Add what the caller does not know, and wrap with `%w`:

```go
if err != nil {
	return fmt.Errorf("get repository file %q: %w", path, err)
}
```

- The message is lowercase, has no trailing punctuation, and does not start
  with "failed to" or "error": the chain already says it failed.
- Use `%w` when callers may match the wrapped error, `%v` when the wrapped
  error is an implementation detail they must not depend on.
- Return `err` unchanged when there is nothing to add.

## Errors callers match

When callers need to tell one error from another, the package exports it:

```go
var ErrNotFound = errors.New("not found")
```

For an error that carries data, export a type:

```go
type ValidationError struct {
	Field string
}

func (e *ValidationError) Error() string {
	return fmt.Sprintf("invalid %s", e.Field)
}
```

Callers match with `errors.Is` and `errors.As`, never by comparing the message:

```go
if errors.Is(err, githubclient.ErrNotFound) {
	// ...
}

var validationErr *config.ValidationError
if errors.As(err, &validationErr) {
	// ...
}
```

Export an error only when some caller matches it. Everything else is a wrapped
error without a name.

## Package boundaries

A package matches the errors of the packages it calls, not of the packages
those call. When A calls B and B calls C, A does not import C to inspect an
error. B translates what A needs to know into its own error.

The same applies to third-party libraries. A package that wraps one translates
the library's errors at that boundary:

```go
_, _, _, err := client.Repositories.GetContents(ctx, owner, repo, path, opts)
var responseErr *github.ErrorResponse
if errors.As(err, &responseErr) && responseErr.Response.StatusCode == http.StatusNotFound {
	return RepositoryFile{}, fmt.Errorf("repository file %q in %s/%s: %w", path, owner, repo, ErrNotFound)
}
if err != nil {
	return RepositoryFile{}, fmt.Errorf("get repository file %q: %w", path, err)
}
```

For Kubernetes API errors, use the matchers of
`k8s.io/apimachinery/pkg/api/errors` (`IsNotFound`, `IsConflict`, …) or
`client.IgnoreNotFound` from controller-runtime.

## Zero values

A function that returns a non-nil error returns the zero value for everything
else, and callers do not use the other values when the error is not nil.

```go
func Load(path string) (Config, error) {
	data, err := os.ReadFile(path)
	if err != nil {
		return Config{}, fmt.Errorf("read config: %w", err)
	}
	// ...
}
```

[errors]: https://pkg.go.dev/errors
