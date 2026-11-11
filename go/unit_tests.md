# Unit Tests

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



### Diff Comparison

Unit tests tend to operate on complex structures. Simply printing two versions
makes it really hard to identify a particular diff causing issues. We want to
encourage the use of a diff library like `cmp`. See also
https://pkg.go.dev/github.com/google/go-cmp/cmp. Example output looks like the
following. Also see the example test where there is made use of `cmp` when
asserting test results.

```
$ go test ./service/cluster/creator
--- FAIL: Test_Service_Cluster_Creator_defaultAWS (0.00s)
    --- FAIL: Test_Service_Cluster_Creator_defaultAWS/case_0_ensures_that_AZ_value_is_defaulted_correctly (0.00s)
        default_test.go:241: want matching

            {creator.Request}.Scaling.Max:
              -: 3
              +: 4

FAIL
FAIL	github.com/giantswarm/api/service/cluster/creator	0.033s
```



### Table Driven Tests

Use table-driven tests when several cases exercise the same code. Each case
runs as a sub-test with [`t.Run`][t-run], named by the case's `name`, so a
single case can be run with `go test -run 'TestClusterID/nil_object'`.

- Test case fields are unexported, and the loop variable is `tc`.
- Errors are asserted with `errors.Is` or `errors.As` (or testify's
  `require.ErrorIs` and `require.ErrorAs`), never by comparing messages.
- Assertions use [`go-cmp`][go-cmp] or [testify's `require`][testify],
  whichever the repository already uses.
- Use `t.Context()` where the code under test takes a context.

Example function to be tested:

```go
var ErrWrongType = errors.New("wrong type")

func ClusterID(obj any) (string, error) {
	cluster, ok := obj.(*Cluster)
	if !ok {
		return "", fmt.Errorf("want *Cluster, got %T: %w", obj, ErrWrongType)
	}
	return cluster.Spec.ID, nil
}
```

Example test:

```go
func TestClusterID(t *testing.T) {
	testCases := []struct {
		name    string
		obj     any
		want    string
		wantErr error
	}{
		{
			name: "valid object",
			obj:  &Cluster{Spec: ClusterSpec{ID: "foobar"}},
			want: "foobar",
		},
		{
			name:    "nil object",
			obj:     nil,
			wantErr: ErrWrongType,
		},
		{
			name:    "wrong type",
			obj:     struct{}{},
			wantErr: ErrWrongType,
		},
	}

	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			got, err := ClusterID(tc.obj)
			if !errors.Is(err, tc.wantErr) {
				t.Fatalf("err = %v, want %v", err, tc.wantErr)
			}
			if diff := cmp.Diff(tc.want, got); diff != "" {
				t.Fatalf("ClusterID() mismatch (-want +got):\n%s", diff)
			}
		})
	}
}
```

[go-cmp]: https://pkg.go.dev/github.com/google/go-cmp/cmp
[t-run]: https://pkg.go.dev/testing#T.Run
[testify]: https://pkg.go.dev/github.com/stretchr/testify/require
