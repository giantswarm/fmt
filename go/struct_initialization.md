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
