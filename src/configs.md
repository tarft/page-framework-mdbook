# Configuration and texts

The new framework includes a new unified configuration system. Each config option lives in a class implementing the `Config` interface.

To set a config, use the syntax `<Config key="...">...</Config>`, where `key` is the class name, in one of the following locations:

- Instance config
- Directory config
- Page XML
- Task XML

TODO: implement config overrides

Due to the ability to set instance/directory/page-wide defaults, some of the use cases of task inputs / attributes should become configuration options in the new framework. For example, the semantics of Turing machines to use is likely to stay consistent across the instance, so it makes sense to expose as a config.

The local value of a config option can be accessed using a controller:

```java
ctrl.getConfig(MyConfig.class)
```

## Texts

Texts are included here because they use the same underlying system.

Put `MyTexts.properties` and `MyTexts_en.properties` files somewhere in `resources`:

```properties
greet=Hello, {0}!
```

Then execute the `Scripts` repo with the following CLI arguments:

```
generate_texts -p=/path/to/iltis/git/base -p=/path/to/iltis/git/tasks
```

This should generate a `MyTexts` class that you can access with a controller as follows:

```java
ctrl.getTexts(MyTexts.class).greet("world")
```

> [!NOTE]
>
> Java's built-in resource bundles are server-only, GWT's resource bundles are client-only. The main motivation to reimplement the text framework from scratch was to be able to access the texts on both. However, there are numerous other benefits.
>
> - The language isn't baked into the permutation, so it can be switched at runtime using the `Language` config option -- one can have multilingual instances with some pages in German and some in English.
> - It's also possible to override texts just like config options, i.e. if instructors don't like a specific wording used in some task, they can change it locally or globally.
>
> The cost is some loss of efficiency (essentially ALL the texts are sent over the wire when a page is loaded), but I'd wager that in current times that doesn't amount to much anymore.
