# Errors

- [Wrapped Errors](#wrapped-errors)
- [Custom Errors](#custom-errors)
- [Matching Errors](#matching-errors)
- [Handling Third Party Library Errors](#handling-third-party-library-errors)

## Wrapped Errors

We use [microkit] to enrich **all** errors with stack traces.

```go
import "github.com/giantswarm/microerror"
```

For errors returned by function call it is fine to just wrap them and return:

```go
return microerror.Mask(err)
```

Or wrap with additional information and return:

```go
return microerror.Maskf(err, "additional information")
```

Additional information should be lowercased:

```go
// This is wrong.
// Additional information should start with lower letter.
return microerror.Maskf(err, "Additional information")
```

**Pros:**

- We always get stack traces when debugging and know immediately where errors
  have been going through.

**Cons:**

- We have to be very consistent with this. The human effort we have to put into
  it is notable but neglectible once familiar with the concept.


## Custom Errors

We only use typed errors. **Don't do this**:

```go
return microerror.New("something bad happened")
```

When creating a new error create a separate file called `error.go` and put
error definition in the file along with *matcher function*.

- error variables are private
- matcher functions are public
- error variables' names end with *Error*
- matcher functions' names are errors' names prefixed with *Is* and with
  *Error* suffix stripped
- error messages are lowercased
- error messages are generic use `microerror.MaskAnyf` to add details
- error messages should follow variable name without *error*

Example:

```go
var invalidConfigError = &microerror.Error{
	Kind: "invalidConfigError",
}

// IsInvalidConfig asserts invalidConfigError.
func IsInvalidConfig(err error) bool {
	return microerror.Cause(err) == invalidConfigError
}

var initializationTimeoutError = &microerror.Error{
	Kind: "initializationTimeoutError",
}

// IsInitializationTimeout asserts initializationTimeoutError.
func IsInitializationTimeout(err error) bool {
	return microerror.Cause(err) == initializationTimeoutError
}
```

Then in code returning error:

```go
return microerror.Maskf(invalidConfigError, "name must not be empty")
```

Dealing with errors from outside packages

We usually want to align with our error handling conventions, so we might encounter errors like this one in Prometheus:

```go
type AlreadyRegisteredError struct {
	ExistingCollector, NewCollector Collector
}

func (err AlreadyRegisteredError) Error() string {
	return "duplicate metrics collector registration attempted"
}
```

Which we can align with our pattern easily like this:

```go
var alreadyRegisteredError = &microerror.Error{
	Kind: "alreadyRegisteredError",
}

// IsAlreadyRegisteredError asserts alreadyRegisteredError.
func IsAlreadyRegisteredError(err error) bool {
	c := microerror.Cause(err)
	_, ok := c.(prometheus.AlreadyRegisteredError)
	if ok {
		return true
	}
	if c == alreadyRegisteredError {
		return true
	}

	return false
}
```

Some anti-patterns:

```go
// This is wrong.
// Error message doesn't match error variable name.
var invalidConfigError = microerror.New("invalid config file")

// This is wrong.
// Error message ends with *error*
var invalidConfigError = microerror.New("invalid config error")

// This is wrong.
// Error message is capitalized.
var invalidConfigError = microerror.New("Invalid config")
```

**Pros:**

- we can test functions returning
- we can read base error message from variable name
- external packages can assert error types
- consistency

**Cons:**

- Doesn't lean nicely to a `switch`/`case` notation

[microkit]: https://github.com/giantswarm/microkit

## Matching Errors

When code flows through a chain of multiple packages a best practise is to work
against the package interfaces. Keeping communication between packages simple
keeps the code simple and makes it easier to understand. Lets assume we have
three packages: A, B and C. A calls B. B calls C. A real life example can be
found in our microservices where the server package calls the endpoint package
and the endpoint package calls the service package. Error handling here should
be done from package to package without skipping a neighbour. When an error is
thrown in C and propagated back to B, B should handle that error and act
accordingly. In some cases B wants to mask its own error depending on the error
it received from C. Then B returns its own error to A, where A can match against
B's error. A should never match against an error of C. As soon as A imports C
for error handling this code smells and we should think about what we are doing
and why.

There are some rules for defining errors we match against:

- `executionFailedError` should never be matched against. I.e.
  `IsExecutionFailed` matcher should never be used. You can think of it as
  a softer version of `panic`.
- All other errors must be matched in at least one place. Be it in the same
  codebase or another (in a library case).

Those rules are established to reduce risk of returning an error which is
handled in an unexpected way. When an error should not be handled or the way of
handling it is yet unclear `executionFailedError` should be returned. This
allows developer to not check whether and how its handled and makes the whole
system simpler overall.

Example:

```go
// pkg.F may return pkg.timeoutError.
err := pkg.F()

// When pkg.F returns pkg.timeoutError, do not mask it blindly but return
// timeoutError from current package instead, if the receiving caller is
// interested in matching a timeout error explicity.
if pkg.IsTimeout(err) {
	return microerror.Maskf(timeoutError, "calling pkg.F")
} else if err != nil {
	return microerror.Mask(err)
}
```

**Pros:**

- Straight and simple design.
- Error handling of complex projects is easy to understand.

**Cons:**

- There is a notable overhead of repeated errors to keep the contract up.

## Handling Third Party Library Errors

When creating libraries using third party libraries under the hood errors
returned by them must be handled as well. Often times those third party
libraries do not support any error handling or the provided error handling is
not sufficient. Our way of dealing with that problem is creating internal error
matchers which are not exposed to the library users and then using them
transform third party errors into [custom errors](#custom-errors) which [can be
matched](#matching-errors). Custom matchers are written in the
`error_internal.go` file. The rest stays as usual.

Example of real world `error_internal.go` file in `githubclient` package:

```go
package githubclient

import "github.com/google/go-github/github"

func isGitHub404(err error) bool {
	if err == nil {
		return false
	}

	errResponse, ok := err.(*github.ErrorResponse)
	if !ok {
		return false
	}

	if errResponse.Response == nil {
		return false
	}

	return errResponse.Response.StatusCode == 404
}
```

Example usage of `isGitHub404` internal error matcher:

```go
fileContent, directoryContent, _, err := underlyingClient.Repositories.GetContents(ctx, owner, repo, path, opt)
if isGitHub404(err) {
	return RepositoryFile{}, microerror.Maskf(notFoundError, "repository file %#q for owner %#q in repository %#q", path, owner, repo)
} else if err != nil {
	return RepositoryFile{}, microerror.Mask(err)
}
```

In the example above GitHub 404 response error is converted to `notFoundError`
matched by `IsNotFound`. Note that this way we are able to include some
operation specific information in the error annotation instead of having what
is provided by the third party library implementor. This makes the debug
process easier in case of error occurrence.

This is also possible that third party error is handled without returning an
error. In this case the error matcher is not exposed and `error.go` file stays
clean. E.g.:

```go
err := thirdParty.Create(obj)
if isAlreadyExists(err) {
	// The thing is already created.
} else if err != nil {
	return microerror.Mask(err)
}
```

In the example above `obj` is created with third party library. When the object
already exists no error is returned even though the third party library returns
one handled with `isAlreadyExists` internal matcher. The benefit here is that
there is no `IsAlreadyExists` matcher exposed to the wrapping library user.
