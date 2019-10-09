# Helm Chart Image Templating

## Templating

When templating container images in charts `values.yaml` should be structured
like this. This allows us to override `image.registry` in some control planes
e.g. AWS China.


```yaml
image:
  registry: quay.io
  name: giantswarm/kube-state-metrics
  tag: v1.8.0
```


