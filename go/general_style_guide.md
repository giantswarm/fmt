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

- We always get stack traces when debugging and know immediately where errors
  have been going through.

**Cons:**

- We have to be very consistent with this. The human effort we have to put into
  it is notable but neglectible once familar with the concept.


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

var initializationTimeoutError = microerror.New("initialization timeout")

// IsInitializationTimeout asserts initializationTimeoutError.
func IsTPRInitTimeout(err error) bool {
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
var alreadyRegisteredError = microerror.New("already registered")

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

### Matching Errors

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

## Struct Initialization

Structs doing a business logic (non-data structures) should be initialized with
corresponding Config structure. The Config structure is a pure data structure.
The Config structure should be validated inside a New function. In case of
being invalid invalidConfigError should be thrown (definition below).

Default values are set in New function initializing the struct.

The Config struct's fields should be separated into two sections: dependencies
and settings. Dependencies should not have default values.

The Config struct is often prefixed with the initializing struct name to avoid
collisions or to bring clarity.

When it makes it more readable (e.g. many structs being created in the same
function) we often initialize those structs in separate scopes (example below).

Definition of invalidConfigError which should be located in error.go file in
the struct's package:

```go
var invalidConfigError = microerror.New("invalid config")

// IsInvalidConfig asserts invalidConfigError.
func IsInvalidConfig(err error) bool {
	return microerror.Cause(err) == invalidConfigError
}
```

Example struct with its initialization Config and New function. In Config
Logger is a dependency field, Name and Order and Timeout are settings fields.
In Struct logger is a dependency field, name and timeout are settings,
fullName is an internal field.

```go
type Config struct {
	Logger     Logger

	Name    string
	Order   int
	Timeout int
}

type Struct struct {
	logger     Logger

	name    string
	timeout int

	fullName string
}

func New(config Config) (*Struct, error) {
	if config.Logger == nil {
		return nil, microerror.Maskf(invalidConfigError, "config.Logger must not be empty")
	}
	if config.Name == "" {
		return nil, microerror.Maskf(invalidConfigError, "config.Name must not be empty")
	}
	if config.Timeout < 0 {
		return nil, microerror.Maskf(invalidConfigError, "config.Timeout must be bigger than or equal to 0")
	}
	if config.Order <= 0 {
		return nil, microerror.Maskf(invalidConfigError, "config.Order must not be less than 1")
	}

	timeout := 0
	{
		if timeout == 0 {
			timeout = 10
		}
	}

	s := &Struct{
		dependency: config.Dependency,
		logger:     config.Logger,

		name:    config.Name,
		timeout: timeout,

		fullName: fmt.Sprintf("%s-%d", config.Name, config.Order),
	}

	return s, nil
}
```

Example initialization:

```go
var logger Logger

func run() error {
	var err error

	var s *Struct
	{
		c := Config{
			Logger: logger,

			Name:    "Helicopter",
			Order:   1,
			Timeout: 15,
		}

		s, err = New(c)
		if err != nil {
			return microerror.Mask(err)
		}
	}
}
```

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


## Unit Tests

Unit tests should be written when it makes sense. Writing tests first often
helps designing new component if its shape is not enforced by framework
requirements. Unit tests also help to verify that state mutating critical logic
behaves correctly in both various success cases and also in error situations.
Simple logic is not that important but sometimes it makes sense to have a simple
test validating correct construction of bigger object so that if one forgets to
pass configuration parameter, the validation error is caught during development
and not in the deployment. 100% code-coverage is not something that should be
targeted. Writing unit test also makes sense when facing a bug that could have
been prevented by having a test for the relevant code in question.

Use table-driven tests instead of separate test methods when possible. Each test
case in table-driven test should be executed as a sub-test with
[t.Run](https://golang.org/pkg/testing/#T.Run) as following output of `go test`
is then easier and one can execute a single specific test case from within table
if needed.

In table-driven tests the fields of test case struct must be unexported as it's
common practice and makes semantically more sense. When iterating test cases,
it's common pattern to use `tc` as variable name for current element.

Description field must start with `case n` where `n` is the index of the test
starting from 0.

Error matchers `func(error) bool` should be used to assert errors. This makes it
easier to match errors returned from other packages.

Suggested ordering for test case struct fields is following:

  1. `name`
  2. Setup config
  3. Input Parameters
  4. Expected output
  5. Validators
  6. Error matchers

All test validations must be included in `t.Run(...) { }` block as technically
each test case in table is a single test.

Example function to be tested:
```go

func ClusterID(obj interface{}) (string, error) {
	customObject, ok := obj.(SpecificType)
	if !ok {
		return "", microerror.Maskf(wrongTypeError, "obj must be SpecificType")
	}

	return customObject.Spec.ID, nil
}
```

Example test:
```go

func Test_ClusterID(t *testing.T) {
	testCases := []struct {
		name              string
		inputObj          interface{}
		expectedClusterID string
		errorMatcher      func(err error) bool
	}{
		{
			name: "case 0: clusterID returned from valid object",
			inputObj: SpecificType{
				Spec: Spec{
					ID: "foobar",
				},
			},
			expectedClusterID: "foobar",
			errorMatcher:      nil,
		},
		{
			name:              "case 1: wrongTypeError returned when nil passed as obj",
			inputObj:          nil,
			expectedClusterID: "",
			errorMatcher:      IsWrongTypeError,
		},
		{
			name: "case 2: wrongTypeError returned when wrong type passed as obj",
			inputObj: struct {
				foo string
				bar int
			}{
				foo: "wrong type",
				bar: -1,
			},
			expectedClusterID: "",
			errorMatcher:      IsWrongTypeError,
		},
	}

	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			clusterID, err := ClusterID(tc.inputObj)

			switch {
			case err == nil && tc.errorMatcher == nil:
				// correct; carry on
			case err != nil && tc.errorMatcher == nil:
				t.Fatalf("error == %#v, want nil", err)
			case err == nil && tc.errorMatcher != nil:
				t.Fatalf("error == nil, want non-nil")
			case !tc.errorMatcher(err):
				t.Fatalf("error == %#v, want matching", err)
			}

			if clusterID != tc.expectedClusterID {
				t.Fatalf("ClusterID == %q, want %q", clusterID, tc.expectedClusterID)
			}
		})
	}
}
```

**Pros:**

- consistent style in tests
- one can run single test cases in table-driven tests
- single test cases identifiers in test run are always in sync

**Cons:**

- description gets normalized when transformed to test name -> search in code
  doesn't match the one in `go test` output.
