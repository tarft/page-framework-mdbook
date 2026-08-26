# Multi-level feedback

Feedback strategies used to be a core part of the framework, and they were "mandatory" in the sense that every server call had to go through a feedback strategy. As many of our tasks don't fit this straightforward mould, I have relegated feedback strategies to an opt-in thing, similar to things like the Constraint API.

> [!NOTE]
> There is now no built-in way to customize feedback strategies in the page XML. The old framework had a whole system for this, but it's very rare that a task actually uses the ability to reorder feedback generators, and in practice there is usually a single "valid" order. Other orderings often crash, misbehave or visually don't look right on the feedback panel. However, through exposing this configuration, the framework heavily restricted the flow of logic on the server, in a way that made more complicated flows impossible to express without bouncing back and forth between server and client.
>
> Also, the entire thing was just plain confusing (to this day I'm not entirely sure what the `break="true"` flag means)
>
> The feedback configuration that is sometimes needed (e.g. enabling/disabling certain feedback levels, or changing generator parameters like e.g. the amount of random models to try in a brute-force equivalence checker) can be done like [any other configuration](./configs.md); see `DescribePropositions` for a demonstration of enabling/disabling feedback levels.

To use feedback strategies, make a class implementing the `FeedbackStrategy` interface, which looks as follows:

```java
public interface FeedbackStrategy<
        C extends Model,
        S extends Model,
        Result extends Serializable
> extends Model {
    void reset();

    Promise<Optional<Result>> next(ServerController<C, S> ctrl);
}
```

Note that feedback strategies are models -- your class needs to store the progress of the strategy somewhere, and all state must be managed by a model.

In your server model, add a `FeedbackStrategyManager` as a model field and initialize it with an instance of your concrete strategy. By calling `FeedbackStrategyManager#advance`, subsequent feedback levels are automatically precalculated and a `ShowMoreMessage` is sent indicating whether the "show more" button should be shown.
    
