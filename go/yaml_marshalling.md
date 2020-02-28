# YAML Marshalling

There are multiple options to marshal yaml in Go but there is no built-in one.
We decided to use the [sigs.k8s.io/yaml](https://github.com/kubernetes/kubernetes/tree/master/vendor/sigs.k8s.io/yaml) library for that purpose.

**NOTE:** The `!!binary` YAML tag does not work with this library. See
https://github.com/kubernetes/kubernetes/tree/master/vendor/sigs.k8s.io/yaml#caveats.

From https://github.com/kubernetes/kubernetes/tree/master/vendor/sigs.k8s.io/yaml#introduction:

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
	FieldA string `json:"fieldA"`                // GOOD
}

type B struct {
	FieldB string `json:"fieldB" yaml:"fieldB"`  // BAD
}
```
