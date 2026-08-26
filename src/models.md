# Models

Models hold all state. For each session, there are two models: the "server model", which is kept on the server, and the "client model", which is synchronized between the client and server.

Models are organized hierarchically. The top-level model represents the state of the entire page, while some of its descendants represent the state of a single task, and further descendants represent the stage of a task's subcomponents.

The synchronization of the client model is done automatically by the framework, and only sends the nodes of the tree that have been changed since the last synchronization.

## Functionality

The following three rules are key to making the framework function:

1. Models must be able to see when their state is mutated.
2. Once a model is "rooted" at a certain path, that path will never point to a different model.
3. Models must provide reflective read/write access to their state and children (this could be done with reflection on the server, but on the client, we have no such luck)

## Existing model classes

You probably never have to write your own model class, because the following three use cases cover all the bases:

### (1) Leaves

`Cell<T>` is a model wrapper around a non-model `T`.

- To access the inner object for read-only purposes, use `T read()`.
- To access the inner object mutably, use `T modify()`.
- To set the inner object, use `void set(T value)`.

(There is also `NullableCell<T>`, where the inner `T` can be `null`. We can't use `Optional` because that isn't serializable.)

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

## When not to model

Not the entire state of the app has to be "modelified", in the sense that you don't need to deeply rewrite all existing classes using these primitives.

For an outside observer, the model classes above handle models and non-models the same: you can put models in a `Cell` (though that is probably rarely useful), and you can put non-models in a `ListModel` or `CompositeModel`. Everything should work as long as (1) no non-model ever contains a model, and (2) no non-model property is ever mutated.

For example, consider the semantics of the types below (where `Person` is not a model):

- `List<Person>`: Completely constant.
- `ListModel<Person>`: You can add/remove/reorder `Person` objects in the list, but never mutate the objects themselves (e.g. invoke `Person#setName`). Changing the name of a person would require exchanging the item at that position with a new `Person` object.
- `ListModel<Cell<Person>>`: You can add/remove/reorder `Person` objects as well as mutate the `Person` objects via `Cell#modify`.
- `List<Cell<Person>>`: Meaningless - a non-model can never contain models.

