# Server-side logic

Server handlers get `ServerController` object, which has the following capabilities:

- Read the state of the client model
- Read/write the state of the server model
- Access [configuration and texts](./configs.md)
- Send [messages](./rpc.md) to the client
- Run analyses

> [!NOTE]
> Notably, the server controller cannot modify the client model. This is because syncing this in a generalized manner is impossible -- if the modifications are sent back to the client, the client may have already made its own modifications in the meantime, and then both sets of changes would have to be reconciled somehow.
>
> Models will throw an exception if you try to modify them anywhere where it's not allowed.

## Analyses

Run analyses as follows:

```java
ctrl.runAnalysis(new MyAnalysis(), params)
```

Analyses get an `AnalysisController`; the only real point of this controller is to schedule other analyses. It doesn't have the capability to access any model state or configuration. This is because analyses run concurrently, so this kind of access would have to be synchronized.

`runAnalysis` returns a `Promise` object, which can be used for any sort of asynchronous workflow. See JS promises. If you just want to emulate the old `generateAnalysisRequests`, you'd do so as follows:

```java
Promise.all(
    ctrl.runAnalysis(new FooAnalysis(), fooParams),
    ctrl.runAnalysis(new BarAnalysis(), barParams)
).then(results -> {
    // results.get(0), etc.
}).catchReject(error -> {
    // handle
})
```

or just `ctrl.runAnalysis(new FooAnalysis(), fooParams).then(result -> { ... })` if running a single analysis.

> [!TIP]
> Promises work just as well on the client. In the `DefinePropositions` task, the `CheckAllAction` first runs a `CheckSingleRequest` for every subtask, and then follows this up with a `CheckConsistencyRequest` once all subtask feedbacks have returned and are all positive.
