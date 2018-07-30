# Log Messages

## Resource Reconciliation

When doing resource reconciliation action we often have to find out the current
state, and apply changes to bring it towards desired state. This often means
calling some API and then emitting debug logs about that. We should try to
align this logs to become more comfortable when reading them and find problems
faster. Also we should aim for them to be understandable not only by developers
but also by SREs, SEs and any other people who look at them and not
necessarily contribute to the codebase.

### Fact Gathering

Usually the reconciling code discovers some facts about the system in order to
figure out what actions it needs to take in order to push the current state
towards the desired state. Those discovering blocks should look like this
whenever possible (the additional comments should be omitted they serve
informational purpose only):

```go
var err error

// Note all messages contain "system fact" which is human readable name for the
// system fact being discovered. Exception is "cancelling resource" message.
var systemFact string
{
	// Start message. Note "finding out" here.
	r.logger.LogCtx(ctx, "level", "debug", "message", "finding out system fact")

	systemFact, err = pkg.FindSystemFact(ctx)
	if IsNotFound(err) {
		// Not found log. Note "did not find" here.
		r.logger.LogCtx(ctx, "level", "debug", "message", "did not find system fact")
		// Reason log should be separate free form log explaining why
		// the system fact can't be found.
		r.logger.LogCtx(ctx, "level", "debug", "message", "waiting for system fact to be created")
		// Canceling resource log should be separate.
		r.logger.LogCtx(ctx, "level", "debug", "message", "canceling resource")
		return nil
	} else if err != nil {
		return microerror.Mask(err)
	}

	// End message. Note "found" here.
	r.logger.LogCtx(ctx, "level", "debug", "message", "found system fact")
}
```

### Ensuring State

After all the facts are gathered there is usually a reconciliation action which
is ensuring the resource is created and up to date or deleted depending on the
observed object event type. This block should look like this whenever possible
(the additional comments should be omitted they serve informational purpose
only):

```go
var err error

// Note all messages contain "managed resource" which is human readable name
// for the resource being ensured.
{
	// Start message. Note "ensured" here. "deletion of" should be present
	// in case of delete event.
	r.logger.LogCtx(ctx, "level", "debug", "message", "ensuring [deletion of] managed resource")

	err = pkg.APICall(ctx, systemFact)
	if IsDeletionInProgress(err) {
		// Did not ensure log. Note "did not ensure" here. "deletion of"
		// should be present in case of delete event.
		r.logger.LogCtx(ctx, "level", "debug", "message", "did not ensure [deletion of] managed resource")
		// Reason log should be separate free form log explaining why
                // the managed resource can't be ensured.
		r.logger.LogCtx(ctx, "level", "debug", "message", "waiting for the API call to finish")
		// Canceling resource log should be separate.
		r.logger.LogCtx(ctx, "level", "debug", "message", "canceling resource")
		return nil
	} else if err != nil {
		return microerror.Mask(err)
	}

	err = pkg.OtherAPICall(ctx, systemFact)
	if IsDeletionInProgress(err) {
		// Fall trough.
	} else if err != nil {
		return microerror.Mask(err)
	}

	// End message. Note "ensured" here. "deletion of" should be present in
	// case of delete event.
	r.logger.LogCtx(ctx, "level", "debug", "message", "ensured [deletion of] managed resource is created")
}
```

### Control Flow

We use `*context` packages for control flow of the reconciliation loop. Here are
log messages associated with specific actions.

```go
r.logger.LogCtx(ctx, "level", "debug", "message", "keeping finalizers")
finalizerskeptcontext.SetKept(ctx)

r.logger.LogCtx(ctx, "level", "debug", "message", "canceling reconciliation")
reconciliationcanceledcontext.SetCanceled(ctx)

r.logger.LogCtx(ctx, "level", "debug", "message", "canceling resource")
resourcecanceledcontext.SetCanceled(ctx)
```

### Resource Intercommunication

We use `controllercontext` package to pass information between resources.
Because of asynchronous nature of resources the information is not available
immediately. If not, the resource should log that and cancel itself.

```go
controllerCtx, err := controllercontext.FromContext(ctx)
if err != nil {
	return microerror.Mask(err)
}

workerASGName := controllerCtx.Status.Drainer.WorkerASGName
if workerASGName == "" {
	r.logger.LogCtx(ctx, "level", "debug", "message", "worker ASG name is not available yet")
	r.logger.LogCtx(ctx, "level", "debug", "message", "canceling resource")
	return nil
}
```
