# YAML Marshalling

There are multiple options to marshal yaml in Go but there is no built-in one.
We decided to use https://github.com/ghodss/yaml library for that purpose.

**NOTE:** The `!!binary` YAML tag does not work with this library. See
https://github.com/ghodss/yaml#caveats.

From https://godoc.org/github.com/ghodss/yaml:

> In short, this package first converts YAML to JSON using go-yaml and then
> uses json.Marshal and json.Unmarshal to convert to or from the struct. This
> means that it effectively reuses the JSON struct tags as well as the custom
> JSON methods MarshalJSON and UnmarshalJSON unlike go-yaml.

There is a good explanation for doing so written down here:
http://web.archive.org/web/20190603050330/http://ghodss.com/2014/the-right-way-to-handle-yaml-in-golang/.

This implies that we do not tag Go structs with `yaml` tags. Only `json` tags
are used by the library.

Example:

```go
type A struct {
	FeildA string `json:"fieldA"`                // GOOD
}

type B struct {
	FeildB string `json:"fieldB" yaml:"fieldB"`  // BAD
}
```
