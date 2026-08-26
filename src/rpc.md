# Client-server communication

## Client &rightarrow; Server

The client controller can send requests:

```java
ctrl.makeRequest(new MyServerRequest());
```

`MyServerRequest` is just a class implementing `ServerRequest`, which is an empty interface.

The request is handled by a server model node that implements `RequestEndpoint`, which should forward it to a [`RequestHandler`](./server.md):

```java
public void declareRequestHandlers(RequestHandlerRegistry<MyServerModel> registry) {
    registry.on(MyServerRequest.class, new MyServerHandler());
}
```

The model node the client controller is attached to determines which model the request arrives at on the server.

## Server &rightarrow; Client

In the new framework, the server isn't restricted to returning one response to a request. Instead, it can send several *messages*, which may be staggered.

```java
ctrl.sendMessage(MyMessage.class);
ctrl.sendMessage(AnotherMessage.class);
```

> [!NOTE]
>
> The main motivation for this is multi-level feedback -- the server needs to send the next feedback and then later a boolean indicating whether more feedback can be requested.
>
> This used to be built into the framework, with designated GWT services, but now can be implemented using the existing primitives.


Messages have an `execute` method with the same signature as that of actions, which gets executed when the message arrives on the client. Most non-framework message classes are conventionally called `...Feedback.java` because the main thing they do is add something to the feedback panel.

## Conflicting requests

The framework guarantees that `ServerRequest`s and `Message`s arrive in a consistent order. However, it does NOT guarantee consistent order between different requests. For example, in the following sequence of events, steps (3) and (4) could happen in either order:

1. Client action triggers Request A
2. Before any server message arrives, another client action triggers Request B
3. Message for Request A arrives back on the client
4. Message for Request B arrives back on the client

This is because Request A may trigger an analysis on the server, during which the server could have time to concurrently handle Request B and send its response back earlier.

However, this can be problematic e.g. when the client triggers two requests that both eventually end up modifiying the same feedback panel. Whichever response arrives last will have the final say on what appears on the panel. Worse yet, if multi-level feedback is used, next-level feedback or the "show more" button could pop up on states of the feedback panel where it's not supposed to. We say that those two requests "conflict".

There are two ways one can try and prevent this:

1. Make it possible to "cancel / invalidate" requests, and make requests cancel all conflicting requests that were previously sent.
2. Prevent conflicting requests from being executed in the first case.

Unfortunately, while (1) would be preferable (since it lets students say "wait, nevermind, I actually wanted *this*" while feedback is calculating), it is very tricky to solve in general. For instance, what if the cancelled request modified some server state (e.g. triggered the task to finish) before the server could be notified of the cancellation?

Therefore, (2) is left as the only realistic option. My recommended way to deal with this is to prevent such "conflicting" requests from occurring at the UI level, i.e. disable "check" buttons while a request is running.

> [!NOTE]
>
> Usually, tasks can make do with just one concurrent request running at a time, but more flexibility may be helpful for some tasks. In `DescribePropositions`, for example, it is perfectly alright if more than one "subtask" is checked at once, but while subtasks are being checked we may not check for consistency and vice versa.

## Implementation details

Currently, the message mechanism is implemented in quite a crude manner where, whenever the client is expecting more server messages, it will poll the server in regular intervals asking for status updates. The framework will, however, bundle `ServerRequest` and `Message` objects together intelligently such that, on each "tick", at most one HTTP request is sent.

The framework guarantees that requests/responses arrive in the order that they are sent by associating them with incrementing IDs and caching mismatched ones until the missing ones arrive. If the cache exceeds a certain limit, the server should discard the session and the client should show an error indicating that the connection was lost (TODO implement).

> [!NOTE]
> One could think about replacing the current implementation with something based on websockets, which is arguably more suitable for this use case, but I don't think that's built into GWT and I'm hesitant to try a [library](https://code.google.com/archive/p/gwt-ws/) in case it introduces even more weird bugs than what we're already dealing with.
