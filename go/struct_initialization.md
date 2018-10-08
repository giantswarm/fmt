# Struct Initialization

Structs implementing business logic should be initialized via a corresponding
constructor which expects a `Config` parameter. The `Config` parameter is a pure
data structure. The `Config` parameter and its fields should be validated inside
the constructor. In case of being invalid an `invalidConfigError` error should
be returned as defined below in an `error.go` file.

```go
var invalidConfigError = microerror.New("invalid config")

// IsInvalidConfig asserts invalidConfigError.
func IsInvalidConfig(err error) bool {
	return microerror.Cause(err) == invalidConfigError
}
```

Our example struct below with its `Config` parameter and constructor which is
represented by the `New` function are explained as follows. In `Config` `Logger`
is a dependency, `Name` and `Order` and `Timeout` are settings. In `Struct`
`logger` is a dependency, `name` and `timeout` are settings, `fullName` is an
internal.

Default values are set within the constructor. The `Config` struct's fields
should be separated into two sections: *dependencies* and *settings*.
Dependencies should not have default values. Settings are any simple data types
not implementing major business logic. The different blocks order their own
fields alphabetically.

```go
// Config intends to provide all necessary dependencies and settings to
// initialize our Struct implementation properly.
type Config struct {
	// Dependencies.
	Logger     Logger

	// Settings.
	Name    string
	Order   int
	Timeout int
}

// Struct implements some business logic in our example.
type Struct struct {
	// Dependencies.
	logger     Logger

	// Internals.
	fullName string

	// Settings.
	name    string
	timeout int
}

// New is the constructor in our example.
func New(config Config) (*Struct, error) {
	// Dependencies.
	if config.Logger == nil {
		return nil, microerror.Maskf(invalidConfigError, "%T.Logger must not be empty", config)
	}

	// Settings.
	if config.Name == "" {
		return nil, microerror.Maskf(invalidConfigError, "%T.Name must not be empty", config)
	}
	if config.Timeout < 0 {
		return nil, microerror.Maskf(invalidConfigError, "%T.Timeout must be bigger than or equal to 0", config)
	}
	if config.Order <= 0 {
		return nil, microerror.Maskf(invalidConfigError, "%T.Order must not be less than 1", config)
	}

	timeout := 0
	{
		if timeout == 0 {
			timeout = 10
		}
	}

	s := &Struct{
		logger:     config.Logger,

		fullName: fmt.Sprintf("%s-%d", config.Name, config.Order),

		name:    config.Name,
		timeout: timeout,
	}

	return s, nil
}
```

When initializing our example `Struct` above, it would look something like the
following.

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
