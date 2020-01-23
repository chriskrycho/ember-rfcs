- Start Date: 2020-01-23
- Relevant Team(s): Ember.js, Learning
- RFC PR: (after opening the RFC PR, update this with a link to it and update the file name)
- Tracking: (leave this empty)

# Improved Ember Registry APIs

## Summary

> One paragraph explanation of the feature.

## Motivation

> Why are we doing this? What use cases does it support? What is the expected
outcome?

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

However, this is less amenable to later iteration, and requires that the implementation be a matter of introspecting on argument order. Supporting namespaces here would require *another* string argument, and functions with multiple arguments of the same type are notoriously easy to misuse. ("Which one goes first, namespace, or type?")

## Unresolved questions

> Optional, but suggested for first drafts. What parts of the design are still
TBD?
