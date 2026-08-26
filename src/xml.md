# XML framework

I implemented a declarative framework for XML parsing, which is used for parsing pages and configuration.

The framework abstracts away the difference between attributes and child elements, which is why attributes are capitalized.

Instead of

```xml
<Person>
  <Name>Dave</Name>
  <Age>69</Age>
</Person>
```

for example, you may equivalently write

```xml
<Person Name="Dave" Age="69" />
```

Parsing is "reversible"; given an object, the framework can serialize it to an XML representation that, when parsed back, will yield the same object.

The framework is designed to be like an XML version of the Jackson JSON library; it can parse arbitrary Java classes and "just work", and annotations are used to customize the behavior.

> [!NOTE]
> The framework is pending a rewrite. Because the "bijective" functionality of the framework is more of a nice-to-have that I currently don't see a use for, I am considering removing it and making it "parse-only". Furthermore, I am considering removing the "attributes = children" equivalence and making it explicit (with annotations saying "make this an attribute" or "make this a child element")
>
> Both of these functionalities were conceived of at a time where I was still experimenting with other framework ideas, and they seem less useful to me now. They both have the disadvantage of making the mechanics harder to understand, and making the XML parsing less flexible.
