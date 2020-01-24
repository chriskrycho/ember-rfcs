- Start Date: 2020-01-23
- Relevant Team(s): Ember.js, Learning
- RFC PR: (after opening the RFC PR, update this with a link to it and update the file name)
- Tracking: (leave this empty)

# Improved Ember Registry APIs

## Summary

Introduce a new, object-based API for all registry APIs; deprecate the current single-string-based registry APIs; and introduce a `schema` property to the resolver to safely support existing resolvers.

## Motivation

> Why are we doing this? What use cases does it support? What is the expected
outcome?

Additionally, there may be some very small performance wins here, since in this schema there is no need to parse a string. This is not considered to be a significant motivator, however, since the tradeoffs between object allocation (memory utilization) and string parsing (CPU utilization) are likely to be largely irrelevant in practice for this API.

## Detailed design

> This is the bulk of the RFC.

> Explain the design in enough detail for somebody
familiar with the framework to understand, and for somebody familiar with the
implementation to implement. This should get into specifics and corner-cases,
and include examples of how the feature is used. Any new terminology should be
defined here.

## How we teach this

> What names and terminology work best for these concepts and why? How is this
idea best presented? As a continuation of existing Ember patterns, or as a
wholly new one?

> Would the acceptance of this proposal mean the Ember guides must be
re-organized or altered? Does it change how Ember is taught to new users
at any level?

> How should this feature be introduced and taught to existing Ember
users?

## Drawbacks

> Why should we *not* do this? Please consider the impact on teaching Ember,
on the integration of this feature with other existing and planned features,
on the impact of the API churn on existing apps, etc.

> There are tradeoffs to choosing any path, please attempt to identify them here.

## Alternatives

> What other designs have been considered? What is the impact of not doing this?

> This section could also include prior art, that is, how other frameworks in the same domain have solved this problem.

### A new String-based API

Instead of providing an object-based API, we could provide a new string-based API. The `lookup` method, then, would have a signature like this:

```ts
interface Owner {
  lookup(type: string, name: string, options: Options);
}
```

This has the advantage of being a smaller diff from today's world (and fewer characters to type):

```diff
- lookup('service:foo')
+ lookup('service', 'foo')
```

vs.

```diff
- lookup('service:foo');
+ lookup({ type: 'service', name: 'foo' })
```

This is a less flexible proposal, and—more importantly—it has a significant downside for usability beyond the simplest case. Supporting namespaces here would require *another* string argument, and functions with multiple arguments of the same type are notoriously easy to misuse. ("Which one goes first, namespace, or type?")

### String-based API as “sugar”

While we prefer to have the object-based API as the *primary* API, we could supply the string-based API outlined above for the simple and most common case, so that this—

```js
lookup('service', 'foo')
```

—would be equivalent to this:

```js
lookup({ type: 'service', name: 'foo'})
```

This could be nice from an ergonomics perspective, and introduces only a small amount of extra complexity in the implementation: the resolver schema still clearly distinguishes between the `lookup('<type>:<name>')` form and `lookup(type, name)`. However, it does require introspection on the arguments passed to the functions.

### Object-based API

For a syntax 

```js
lookup({ service: 'foo' })
```

While this is nice from the perspective of the consumer of the API, it muddies the API substantially and makes implementation more complex and with much worse performance: the lookup would have to iterate over all keys on the object passed every time, mapping those keys to the *known* identifer interface and then checking the others against the registry.

Implementer concerns should not be *primary*, but they are important, and here they have performance implications that, while not dramatic, can easily be avoided by other proposals.

## Unresolved questions

- What is the right name for the parameter and interface for the lookup? `identifier` is a reasonable choice, but does overlap with the notion of identifiers from Ember Data.

## Appendix: TypeScript

This design is motivated in part by a desire for the API to better support type-safe TypeScript usage. If we were to ship this, we would accompany it with the following types at DefinitelyTyped:

```ts
interface TypeRegistry {
  service: ServiceRegistry;
  controller: ControllerRegistry;
}

interface LookupOptions {
  singleton?: boolean;
}

interface Identifier<
  Type extends keyof TypeRegistry,
  Name extends keyof TypeRegistry[Type],
> {
  type: Type;
  name: Name;
  namespace?: string;
}

interface Owner {
  lookup<
    Type extends keyof TypeRegistry,
    Name extends keyof TypeRegistry[Type],
  >(identifier: Identifier<Type, Name>, options?: Options): TypeRegistry[Type][Name];
}
```