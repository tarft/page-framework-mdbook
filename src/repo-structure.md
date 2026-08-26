# Repository structure

What previously was `tool` now consists of three repositories: `base`, `tasks` and `runner`. In addition there has been some change in the existing repositories.

## `base`

Contains the educational framework powering Iltis. Only imports `utils`.

## `tasks`

The main repository, importing `base`. Contains (1) task implementations and (2) components shared by multiple tasks.

Rule of thumb: If something is generic enough to be used outside of the theoretical CS context (e.g., could be used to make a tool for teaching Klingon), it belongs in `base`. Otherwise, it should be in `tasks`.

Structured as follows:

```
- client/shared/server
  - impl
    - fola
      - ...
      - ...  (one directory per task)
      - ...
    - logic
      - ...
      - ...  (one directory per task)
      - ...
    - misc
      - ...
      - ...  (one directory per task)
      - ...
  - general
    - ...
    - ...  (shared components) 
    - ...
```

Tasks are independent in the sense that no task ever imports things from another task's directory.

## `runner`

The actual "executable", which defines all the GWT services and stitches all the parts provided by `tasks` together, but doesn't contain any interesting functionality of its own.

Basically, `runner` and `base` are the two unmoving "sandwich buns" that provide the framework, with `tasks` as the "meat" between them.

## `utils` and libs

Now also obey the `client`/`shared`/`server` convention demanded by GWT modules.

There are some new things in `utils` that have cross-cutting importance:

- `UniqueId` and `LocalId` provide opaque IDs; these are useful to avoid shared / circular references (which are bad for serialization). `UniqueId`s represent UUIDs, while `LocalId`s are incrementing counters.
- `Promise` is an async primitive that loosely follows the JS Promise API, and is used for asynchronicity in the new framework, on both client and server.
- `Widgetable` is an interface for serializable classes that can be converted into GWT widgets. This is used principally for feedback. 
- The `Text` class is to be used instead of `String` whenever handling raw HTML. It implements the `Widgetable` interface.
- The new XML framework, which lives in `utils.server.spec`, replaces both `Parsable` and our hand-written XML data loaders. It is designed to be declarative without having the limitations of `Parsable`. In addition, it's bidirectional; it can serialize objects *to* XML without requiring additional effort. There's no use for that yet but it's a good property to have.

Other than restructuring (`client`/`shared`/`server`), the libs are unchanged.

> [!NOTE]
> I am doing the restructuring incrementally, i.e. pulling things into `shared`/`server` as I need them.

## `iltiscss`

Now split between resources `base` and `tasks`, instead of being its own module. Run `mvn generate-resources` in those repos to update styles.

> [!NOTE]
> The splitting of tool means we can no longer use some features of Sass to their full effect. While it's possible to move the Sass files from `base` and `tasks` to one location at build and use a single Sass compilation for both, IntelliJ doesn't understand that that's what we're doing, so Sass variables / mixins in `base` will be shown as "not found" in `tasks`. So, I just elected to have separate Sass compilations, because that seemed like the cleanest solution.
> 
> Fortunately, I think Sass variables / mixins are a menace upon our codebase anyway, because unlike native CSS variables / HTML classes they don't show up in the compiled output, so it's hard to tell where some applied style actually comes from.
> 
> (In theory, with scoped style rules now being stable all major browsers, I don't really think we need Sass at all, but there's no harm in keeping it around either)

> [!TIP]
> Here's a script to convert our old Sass variables to native CSS ones:
>
> ```js
> // usage:
> //     node desass.js [path] [target]
> //     node desass.js [path]           # Replaces the old file
>
> const fs = require("fs");
> 
> function replaceColors(content) {
>   return content.replace(
>     /colour\("([^"]+)",\s*"([^"]+)"\)/g,
>     (_, p1, p2) => `var(--iltis-${p1}-${p2})`,
>   );
> }
> 
> function replaceOtherFunctions(content) {
>   return content.replace(
>     /([\w-]+)\("([^"]+)"\)/g,
>     (_, p1, p2) =>
>       `var(--iltis-${p1}-${p2.replace(/[A-Z]/g, "-$1").toLowerCase()})`,
>   );
> }
> 
> let s = fs.readFileSync(process.argv[2], "utf-8");
> s = replaceColors(s);
> s = replaceOtherFunctions(s);
> fs.writeFileSync(process.argv[3] || process.argv[2], s);
> ```


