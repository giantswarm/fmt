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
https://godoc.org/github.com/google/go-cmp/cmp. Example output looks like the
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
	customResource, ok := obj.(SpecificType)
	if !ok {
		return "", microerror.Maskf(wrongTypeError, "obj must be SpecificType")
	}

	return customResource.Spec.ID, nil
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
		t.Run(strconv.Itoa(i), func(t *testing.T) {
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
				t.Fatalf("\n\n%s\n", cmp.Diff(clusterID, tc.expectedClusterID))
			}
		})
	}
}
```

**Pros:**

- consistent style in tests
- one can run single test cases in table-driven tests
- single test cases identifiers in test run are always in sync
- useful diff comparison
- run `go test ./... -run Test_Foo/5` to only execute a single test

**Cons:**

- description gets normalized when transformed to test name -> search in code
  doesn't match the one in `go test` output.
