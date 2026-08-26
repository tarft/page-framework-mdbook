# Client-side logic

All computation on the client is kickstarted by the view launching an action. Actions have an `execute` method through which they receive a `ClientController` object. Using this `ClientController`, actions can do the following:

- Read/write the state of the client model
- Access [configuration and texts](./configs.md)
- Send [server requests](./rpc.md)

`ClientController`s (and the other controllers we will see later) are attached to a model node, but expose an `ctrl.at(Model)` method through which one can cheaply obtain a controller for a different model node.

> [!NOTE]
> A more "OOP-brained" designer would probably make all these capabilities methods on `Action`, etc., so that you can just write `makeRequest(...)` instead of `ctrl.makeRequest(...)`.
>
> Not so me -- the framework includes virtually no abstract base classes (only `CompositeModel` and `Texts`, both to save on boilerplate) and does everything via interfaces. This IMO makes the code a lot more clean. These controllers are effectively "framework handles" that can be passed around to methods to let them do framework stuff.

## Events

By implementing the `EventEndpoint` interface, nodes of the client model can react to certain events, such as:

- An action getting executed on them or any descendant node
- A [message](./rpc.md) being received by them or any descendant node
- A (logical) timestep passing (i.e., any state changed anywhere)
- ...

The event handler gets passed a `ClientController`, and so has the same capabilities as an action.

While it's recommended to use actions and messages for most logic, these events allow hooking directly into the framework, which is sometimes necessary.

## Continuous ("fine-grained") actions

In the new framework, EVERY user action that impacts the UI state should be an `Action`. This includes things like "dragging a node" or "typing in a text field", which are "continuous", not "discrete".

For keeping action logs / the task history / ... compact, the current action can be conditionally merged to the previously executed action. Presumably, one would separate events like "typing in a text field" by "cutting" at every n-second interval or after m seconds of inactivity.
