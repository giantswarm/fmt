# Dockerfile

### Base Images

- We use [alpine-linux] whenever possible.
- We don't use the `latest` tag unless there is a very good reason as its a bad
practice.
- We use the latest minor release of [alpine-linux] e.g. `alpine:3.9`. 

### Go Images

- Most of our Docker images package Go binaries.
- We try to install the minimum number of packages. Usually only
`ca-certificates` is necessary.
- We just add the binary and set the [entrypoint] to the binary.
- Here is an example Dockerfile for [app-operator] in this case. 

```Dockerfile
FROM alpine:3.9

RUN apk add --no-cache ca-certificates

ADD ./app-operator /app-operator

ENTRYPOINT ["/app-operator"]
```

[alpine-linux]: https://alpinelinux.org/
[app-operator]: https://github.com/giantswarm/app-operator/blob/master/Dockerfile
[entrypoint]: https://docs.docker.com/engine/reference/builder/#entrypoint
