# Models

Models hold all state. For each session, there are two models: the "server model", which is kept on the server, and the "client model", which is synchronized between the client and server.

Models are organized hierarchically. The top-level model represents the state of the entire page, while some of its descendants represent the state of a single task, and further descendants represent the stage of a task's subcomponents.

The synchronization of the client model is done automatically by the framework, and only sends the nodes of the tree that have been changed since the last synchronization.

## Functionality

The following rules are key to making the framework function:

1. Models must be able to track when/how their state is mutated.
2. Once a model is "rooted" at a certain path, that path will never point to a different model.
3. Models must provide reflective read/write access to their state and children (this could be done with reflection on the server, but on the client, we have no such luck).

In short, models **provide a familiar mutation-like API to state that internally is "append-only"**.

## Existing model classes

You probably never have to write your own model class, because the following three use cases cover all the bases:

### (1) Leaves

`Cell<T>` is a model wrapper around a non-model `T`.

- To access the inner object for read-only purposes, use `T read()`.
- To set the inner object, use `void set(T value)`.

(There is also `NullableCell<T>`, where the inner `T` can be `null`. We can't use `Optional` because that isn't serializable.)

> [!CAUTION]
> The inner object may never be mutated. This is because the task history saves "snapshots" of how the model used to be, and for `Cell` these are just the inner object itself. If you mutate the inner object, that also impacts the old history entries.
>
> It would be possible to allow internal mutation with the caveat that (1) the responsible model is notified of the mutation (e.g. with a method like `Cell#modify`, with the same semantics as `Cell#read` with the only difference that it marks the model as modified) and (2) the objects are deep-cloned for the task history. In fact, this is how my earlier design worked. However, note that this just offloads the responsibility of deep-cloning to the framework, when you could just as well clone the object and then do your mutations yourself in the rare case that you need to do such I thing.
>
> There is of course no requirement that the inner objects physically CAN'T be mutated. We could go to the effort of providing our own immutable wrappers of Java's standard collections, but that ends up being very tedious because you have to convert between them and the standard collections all the time, so I don't think efforts like `ImmutableList` are worth it at all.

### (2) Structures

`CompositeModel` is an abstract class that can be overridden to make a model that has no inherent state of its own, but has models as its fields. This is like how you would use a struct/class to bundle pieces of data together.

To satisfy the tenets some boilerplate is necessary. Every model child has to be registered as follows:

```java
private Cell<FeedbackType> type = register("type", FeedbackPanelModel::getType, new Cell<>(FeedbackType.NEGATIVE));
public Cell<FeedbackType> getType() {
    return type;
}
```

- The first argument (a unique string label) is used as the model path segment
- The second argument (a getter function) is used to form the association between the field and the model structure
- The third argument (an initial value) is just passed as the return value of `register` -- it can be `null` as long as you definitively initialize the field somewhere else at initialization, e.g. in the constructor

### (3) Collections

`ListModel<T>` and `MapModel<T>` are model-ified versions of Java's `List` and `Map` collections, where items / values can be models.

Because of the rule that model paths can't be changed, an opaque incrementing identifier is used for the path segment instead of the index / key.

TODO: Provide some way for models to be marked as "inactive/irrelevant". Removed entries can't be fully discarded, since this would break undo, but the framework could "garbage-collect" after they are sent to the server (and entered into the DB), and ask the server for the required info when the user requests to roll back that far.

## When not to model

Not the entire state of the app has to be "modelified", in the sense that you don't need to deeply rewrite all existing classes using these primitives. However, the parts that should

For an outside observer, the model classes above handle models and non-models the same: you can put models in a `Cell` (although this is redundant, since you can never call `Cell#set` after this due to the rule that model paths can't be replaced). More usefully, you can put non-models in a `ListModel` or `CompositeModel`. Everything should work as long as (1) no non-model ever contains a model, and (2) no non-model property is ever mutated.

For example, consider the semantics of the types below (where `Person` is not a model):

- `List<Person>`: Completely constant.
- `ListModel<Person>`: You can add/remove/reorder `Person` objects in the list, but never mutate the objects themselves (e.g. invoke `Person#setName`). Changing the name of a person would require exchanging the item at that position with a new `Person` object.
- `ListModel<Cell<Person>>`: The same capabilities as `ListModel<Person>`, just with one more layer of indirection; to replace an entry you can either use `ListModel#set` or `Cell#set`.
- `List<Cell<Person>>`: Invalid - a non-model can never contain models.

Even though combinations like `Cell<SomeModel>` or `ListModel<Cell<T>>` have redundant semantics and should therefore be avoided as compile-time types (one might as well use `SomeModel` and `ListModel<T>`, respectively), they still work fine. Therefore, you can write generic code that puts some `T` into a `Cell` or `ListModel` without worrying about what `T` might possibly be at runtime.
