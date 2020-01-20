# CircleCI best practices

## Making steps self-explanatory

CircleCI offers a way to make a step in a CI config or org self-explanatory. We SHOULD always use this to make the config more self-explanatory.

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
