# Variable Names

Here we want to provide suggestions and best practises about variable names we
tend to use in our code. Aligning with the suggestions below makes our code
easier to scan and code in general easier to write because it is a
no-brainer how to call things, so we can concentrate on what actually matters.

## Controller Context

In `operatorkit` resource implementations, we sometimes work with
`controllercontext` package. We store data extracted from this context in
variable named `cc`.

```go
cc, err := controllercontext.FromContext(ctx)
if err != nil {
	return nil, microerror.Mask(err)
}
```

## Custom Resource

In `operatorkit` resource implementations, we take the parameter `obj` and have
to assert the proper type the resource actually works with. The variable name
we want to use here is `cr`.

```go
func (r *Resource) EnsureCreated(ctx context.Context, obj interface{}) error {
	cr, err := key.ToCustomResource(obj)
	if err != nil {
		return nil, microerror.Mask(err)
	}

	...
}
```
