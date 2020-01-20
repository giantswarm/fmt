# CircleCI best practices

## Making steps self-explanatory

Always set the `name` of your steps to something that explains what it does.

### Don't

```yaml
  steps:
    - run: [ -z ${CIRCLE_TAG} ] && echo -n 'giantswarm-operations-platform-test-catalog' > .app_catalog_name || echo -n 'giantswarm-operations-platform-catalog' > .app_catalog_name
```

### Do

```yaml
  steps:
    - run:
        name: Set target catalog name in temp file
        command: [ -z ${CIRCLE_TAG} ] && echo -n 'giantswarm-operations-platform-test-catalog' > .app_catalog_name || echo -n 'giantswarm-operations-platform-catalog' > .app_catalog_name
```
