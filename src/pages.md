# Task I/O

There are three ways a task interfaces with its environment:

- Task inputs, received from other tasks
- Task data, specified in XML
- Task outputs, passed on to other tasks

(There is also [configuration](./configs.md), which a task may read)

The `TaskSpecifier`'s job is to define:

- the class used to serialize/deserialize the task data
- how to initialize the tasks, given the task data and inputs
- how to generate outputs from the task's finish state.

> [!NOTE]
> In this new framework, there is just one, completely opaque, XML specification a task is loaded from, which is completely independent from the task inputs. Contrast this with the old framework, where there were just task inputs, and they could be either provided or specified in XML.
>
> The old system had its advantages; the situation where an input exactly specifies a part of the task data is quite common and natural. However, there were also many cases where this doesn't work, and those had to be clumsily worked around. Often, when a new way of linking tasks or some other kind of generalization was retroactively needed, the task inputs had to be changed or a "dummy input" had to be hacked in. The new system is a lot more flexible to allow for expanding functionality without breaking existing pages.

## Finishing tasks

Call `TaskServerModel.finishTask` anywhere on the server to finish the task. This will trigger.

Just like feedback strategies, the "task unlock logic" isn't really a core part of the framework. It should be possible with
