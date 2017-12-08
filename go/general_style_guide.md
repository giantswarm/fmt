# Go Style Guide

> [Gofmt's style is no one's favorite, yet gofmt is everyone's favorite.](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=8m43s)
>
> -- Rob Pike

It is important to follow this guide. It makes review process easier and
faster. It also makes our codebase consistent, easier to navigate, and
understand.

## Errors


### Wrapped Errors

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

- TBD

**Cons:**

- TBD


### Custom Errors

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
var invalidConfigError = microerror.New("invalid config")

// IsInvalidConfig asserts invalidConfigError.
func IsInvalidConfig(err error) bool {
	return microerror.Cause(err) == invalidConfigError
}

var initializationTimeout = microerror.New("initialization timeout")

// IsInitializationTimeout asserts initializationTimeoutError.
func IsTPRInitTimeout(err error) bool {
	return microerror.Cause(err) == initializationTimeout
}
```

Then in code returning error:

```go
return microerror.Maskf(invalidConfigError, "name must not be empty")
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

## Imports

The import block should be split into 3 separate blocks in this order.

- Native packages.
- Packages not in the current project.
- Packages in the current project.

```go

import (
	"context"

	jsonpatch "github.com/evanphx/json-patch"
	"github.com/giantswarm/microerror"

	"github.com/giantswarm/cluster-service/service/cluster/storage"
)
```

**Pros:**

- consistency
- imports are easier to read

**Cons:**
