# Time-travelling

TODO: Implement, further think about undo/redo

On every server request (as well as in regular intervals even without server requests -- as long as the user hasn't been totally inactive during that time), the new client state modifications are sent to the server. The server then makes log entries in the database that serve a dual purpose -- they let us analyze the student data, and they let us recover old state, which I call "time-travelling".

There are three ways in which one might time-travel:

- Undo/redo
- Page restoration
- Debug time-travelling (in the debug panel, click on a point in time to see a static view of what the page was like at that time. The same principle could also be used to "replay" a student's task-solving process in real time, which I imagine is quite powerful for understanding how students solve tasks)

These all use the same underlying principles.

## Idle points

For simplicity, we first look at the case where we want to take a "snapshot" of the entire state of the page and reinstate it later. A crude version of undo/redo could be implemented this way, but this isn't the best way; see below.

At any point in time, there may be pending server requests / messages. When we time-travel, we can't easily handle these. To be specific, there are two sources of pending server requests / messages:

* Those at the "source point" from where the time-travelling was initiated (e.g. the state where we click "undo")
* Those at the "target point" where we wish to time-travel to (e.g. the state that "undo" should get us to)

Neither of those are allowed to have any pending communication, i.e. they should be "idle" points. (For debug time-travelling the idleness of "target" doesn't matter since it's unresponsive anyways)

Thus, when a user requests to reinstate an old session, we actually restore the last "idle" point, not the last point we recorded. (This makes sense -- it would be confusing to the user, if they happened to disconnect while a server request was pending, to revisit the page and see the same spinners still there.)

## Local time-travelling -- Undo/redo

Transferring the idle-point principle to undo/redo, it follows that undo/redo can only be executed in idle points, and non-idle points are skipped when considering which point to restore the state to.

The special thing about undo/redo is that it's scoped to a certain part of the state (e.g., a single task). We can keep separate undo/redo histories for unrelated tasks, as long as each timestep we skip only has modifications in that part of the state. As soon as a timestep has modifications outside the managed scope, we must report that undoing/redoing is no longer possible.

> [!NOTE]
> Technically this also goes for READING, not just modifications. For instance, imagine we have task A and task B which each manage some state $S_A$ and $S_B$, both initially "foo". Furthermore we have two actions, $A_A$ and $A_B$. The effect of $A_A$ is to set $S_A$ to "bar", while the effect of $A_B$ is to set $S_B$ to the current value of $S_A$. Now watch what happens when we first execute $A_A$, then $A_B$, and then undo $A_A$. We get $S_A$ = "foo" and $S_B$ = "bar", which is a state configuration that we could ordinarily never reach without undo/redo. However, the chance of something like this being relevant is hopefully rather slim...
