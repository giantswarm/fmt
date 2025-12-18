# Imports

Imports must be sorted in the order decided by the `golang.org/x/tools/cmd/goimports` tool.

The import block should be split into 3 separate blocks in this order:

1. Native packages.
2. Packages not in the current project.
3. Packages in the current project.

```go

import (
	"context"

	jsonpatch "github.com/evanphx/json-patch"
	"github.com/giantswarm/microerror"

	"github.com/giantswarm/cluster-service/service/cluster/storage"
)
```

**Pros:**

- consistency
- imports are easier to read

**Cons:**

