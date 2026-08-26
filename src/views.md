# Views

In our old frameworks, views contained a lot of busy logic that was required to make the task execute properly. Now, views are essentially separable from the rest of the framework; in principle, tasks can be run on the server without the view by just providing a list of actions to run. (This could be helpful for mocking tasks as part of an automated test setup)

Views can do three things:

1. Execute actions. Views are attached to a model node (for example, the `XyzTaskView` is attached to the `XyzTaskClientModel`), and the action will be dispatched relative to that node.
2. React to model changes. Whenever the client model changes, views have an opportunity to update their own UI to match the state of the model. Views can check if any model node's state has been modified since last refresh, and surgically update only the parts of the DOM that require it. If this weren't possible, the entire page would have to be rebuilt on every user action.
3. Register other views (and deregister them, if necessary, to avoid leaks).

Since the view's state is completely dependent on the model's state, it is possible to fully construct a view by just being given the model. The applications for this are listed in the [time travelling chapter](./time-travelling.html).

> [!NOTE]
> 
> The "all state is sourced from the model" requirement obviously requires a major reorganization of our widgets (e.g. the graph widget) and may also not play well with third-party libraries. For example, the `DescribePropositions` task uses a reorderable list that was previously implemented with the Sortable.js library, but Sortable.js is stateful and AFAIK doesn't give you this kind of access to its state. So, by the time I properly implement the drag-and-drop, I might have to implement this stuff from scratch and/or somehow hack it to work with Sortable.
>
> Good news for the text editor widget though: CodeMirror is very much built on this principle, so it should pose no issues to adapt to this paradigm.
