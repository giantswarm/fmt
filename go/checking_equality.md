# Checking Object Equality

When checking if 2 Kubernetes structs are equal, it is advised to use `apiequality.Semantic.DeepEqual` from https://github.com/kubernetes/apimachinery/blob/master/pkg/api/equality/semantic.go

When checking if 2 go structs are equal, it is advised to use `reflect.DeepEqual`
