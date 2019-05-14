# Panic

Sometimes we have to `panic` or exit the program with a status code. We should
do that in as few places as possible. Common pattern here is to make a helper
function returning an error and handling all errors returned by the helper
function with `panic` or `os.Exit` in a single place.



## Example 1 - Public Function

We can have a public function `func Run()` which because of reasons can not
return an error.

The handling should look like this:

```go
func Run() {
	err := run()
	if err != nil {
		panic(fmt.Sprintf("%#v", err))
	}
}

// run is a helper function allowing Run to handle panic in a single place.
func run() error {
	// Do the thing with usual error handling.
}
```



## Example 2 - Private Function

We can have a private function `func main()` which because of obvious reasons
can not return an error.

The handling should look like this:

```go
func main() {
	err := mainE()
	if err != nil {
		fmt.Fprintf(os.Stderr, "%#v", err)
		os.Exit(1)
	}
}

// mainE is a helper function allowing main to handle os.Exit in a single place.
func mainE() error {
	// Do the thing with usual error handling.
}
```
