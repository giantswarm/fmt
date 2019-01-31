# Log Messages



## General Guidelines

- We use `%#q` for string formatting in our log & error messages. This prevents
  from escaping when they are embedded in JSON string value.
- The `action` key is useful when filtering logs for a given action. Following
  rules apply:
    - Value of `action` key never changes in scope of action.
    - First `message` for the `action` must be `start`.
    - Last `message` for the `action` can be one of:
        - `canceling reconciliation`
        - `canceling resource`
        - `end`
        - `keeping finalizers`
    - All `message`s in between are free form text.



## Resource Reconciliation

When doing resource reconciliation action we often have to find out the current
state, and apply changes to bring it towards desired state. This often means
calling some API and then emitting debug logs about that. We should try to align
these logs to become more comfortable when reading them so we can find problems faster.
Also we should aim for them to be understandable not only by developers but also
by SREs, SEs and any other people who look at them and not necessarily
contribute to the codebase.



### Control Flow

We use `*context` packages for control flow of the reconciliation loop. Here are
log messages associated with specific actions.

```go
r.logger.LogCtx(ctx, "level", "debug", "action": "...", "message", "keeping finalizers")
finalizerskeptcontext.SetKept(ctx)

r.logger.LogCtx(ctx, "level", "debug", "action": "...", "message", "canceling reconciliation")
reconciliationcanceledcontext.SetCanceled(ctx)

r.logger.LogCtx(ctx, "level", "debug", "action": "...", "message", "canceling resource")
resourcecanceledcontext.SetCanceled(ctx)
```



### Resource Intercommunication

We use the `controllercontext` package to pass information between resources.
Because of the asynchronous nature of resources the information is not available
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



### Error Handling

At the root of error handling code when there is no error to be returned to the
original caller, errors have to be logged. We try to format such logs in the way
as described in the example below.

```go
s.logger.LogCtx(ctx, "level", "error", "message", "failed to collect metrics", "stack", fmt.Sprintf("%#v", microerror.Mask(err)))
```

The fact gathering statement expresses what happens and in case the action fails
`failed` is prepended to it in the error log message. The additional `stack`
field is added to get some tracing information from the carried error.

```go
s.logger.Log("level", "debug", "action", "collecting metrics", "message", "start")

for _, c := range s.collectors {
	err := c.Collect(ch)
	if err != nil {
		s.logger.LogCtx(ctx, "level", "error", "action", "collecting metrics", "message", fmt.Sprintf("collector %#q failed", c.name), "stack", fmt.Sprintf("%#v", microerror.Mask(err)))
	}
}

s.logger.Log("level", "debug", "action", "collecting metrics", "message", "end")
```



### Fact Gathering

Usually the reconciling code discovers some facts about the system in order to
figure out what actions it needs to take in order to push the current state
towards the desired state. Those discovering blocks should look like this
whenever possible. In the example below the system fact is list of ELBs in AWS
API.

```go
var err error

var elbs []aws.ELB
{
	r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("finding ELBs with name prefix %#q", prefix), "message": "start")

	r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("finding ELBs with name prefix %#q", prefix), "message": "listing all ELBs")

	elbs, err = aws.ListELBs(ctx)
	if aws.IsNotFound(err) {
		// Fall trough.
	} else if err != nil {
		return microerror.Mask(err)
	}

	r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("finding ELBs with name prefix %#q", prefix), "message": fmt.Sprintf("filtering prefixed ELBs from total %d", len(elbs)))

        elbs, err = filterELBs(elbs, prefix)
	if err != nil {
		return microerror.Mask(err)
	}

	r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("finding ELBs with name prefix %#q", prefix), "message", fmt.Sprintf("found %d ELBs", len(elbs)))

	r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("finding ELBs with name prefix %#q", prefix), "message": "end")
}
```



### Reconciling State

After all the facts are gathered there is usually a reconciliation action which
is ensuring the resource is created and up to date or deleted depending on the
observed object event type. This block should look like this whenever possible.

```go
var err error

{
	r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("deleting ELBs with name prefix", prefix), "message": "start")

	for _, elb := range elbs {
		r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("deleting ELBs with name prefix", prefix), "message": fmt.Sprintf("deleting ELB %#q", elb.Name))

		err = aws.DeleteELB(ctx, elb)
		if pkg.IsOperationInProgress(err) {
			r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("deleting ELBs with name prefix", prefix), "message": fmt.Sprintf("ELB %#q deletion is in progress", elb.Name))
			r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("deleting ELBs with name prefix", prefix), "message": "keeping finalizers")
			finalizerskeptcontext.SetKept(ctx)
		} else if err != nil {
			return microerror.Mask(err)
		}
	}

	r.logger.LogCtx(ctx, "level", "debug", "action", fmt.Sprintf("deleting ELBs with name prefix", prefix), "message": "end")
}
```
