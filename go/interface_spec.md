# Interfaces

When providing an interface for a package, it is convention to do this in a `spec.go` file. 

Interfaces follow a naming convention of being ‘agent noun suffixes’; meaning they’ll end in either `or`, `er` to describe the behaviour. For example, a `write` package, should fulfil a `Writer` interface. A `collect` package, should fulfil a `Collector` interface.

Interfaces are best designed small and simple.

>Well designed interfaces are more likely to be small interfaces; the prevailing idiom is an interface contains only a single method. It follows logically that small interfaces lead to simple implementations, because it is hard to do otherwise. Which leads to packages comprised of simple implementations connected by common behaviour. - _Dave Cheney_

One of the most concrete examples of a well defined interface is the `io.Writer`. A simple, but wildly powerful and implemented interface.
