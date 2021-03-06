<!--- Copyright (C) 2009-2013 Typesafe Inc. <http://www.typesafe.com> -->
# Action composition

This chapter introduces several ways of defining generic action functionality.

## Custom action builders

We saw [[previously|ScalaActions]] that there are multiple ways to declare an action - with a request parameter, without a request parameter, with a body parser etc.  In fact there are more than this, as we'll see in the chapter on [[asynchronous programming|ScalaAsync]].

These methods for building actions are actually all defined by a trait called [`ActionBuilder`](api/scala/index.html#play.api.mvc.ActionBuilder), and the [`Action`](api/scala/index.html#play.api.mvc.Action$) object that we use to declare our actions is just an instance of this trait.  By implementing your own `ActionBuilder`, you can declare reusable action stacks, that can then be used to build actions.

Let’s start with the simple example of a logging decorator, we want to log each call to this action.

The first way is to implement this functionality in the `invokeBlock` method, which is called for every action built by the `ActionBuilder`:

@[basic-logging](code/ScalaActionsComposition.scala)

Now we can use it the same way we use `Action`:

@[basic-logging-index](code/ScalaActionsComposition.scala)
 
Since `ActionBuilder` provides all the different methods of building actions, this also works with, for example, declaring a custom body parser:

@[basic-logging-parse](code/ScalaActionsComposition.scala)


## Composing actions

In most applications, we will want to have multiple action builders, some that do different types of authentication, some that provide different types of generic functionality, etc.  In which case, we won't want to rewrite our logging action code for each type of action builder, we will want to define it in a reuseable way.

Reusable action code can be implemented by wrapping actions:

@[actions-class-wrapping](code/ScalaActionsComposition.scala)

We can also use the `Action` action builder to build actions without defining our own action class:

@[actions-def-wrapping](code/ScalaActionsComposition.scala)

Actions can be mixed in to action builders using the `composeAction` method:

@[actions-wrapping-builder](code/ScalaActionsComposition.scala)

Now the builder can be used in the same way as before:

@[actions-wrapping-index](code/ScalaActionsComposition.scala)

We can also mix in wrapping actions without the action builder:

@[actions-wrapping-direct](code/ScalaActionsComposition.scala)

## More complicated actions

So far we've only shown actions that don't impact the request at all.  Of course, we can also read and modify the incoming request object:

@[modify-request](code/ScalaActionsComposition.scala)

> **Note:** Play already has built in support for X-Forwarded-For headers.

We could block the request:

@[block-request](code/ScalaActionsComposition.scala)

And finally we can also modify the returned result:

@[modify-result](code/ScalaActionsComposition.scala)

## Different request types

The `ActionBuilder` trait is parameterised to allow building actions using different request types.  The `invokeBlock` method can translate the incoming request to whatever request type it wants.  This is useful for many things, for example, authentication, to attach a user object to the current request, or to share logic to load objects from a database.

Let's consider a REST API that works with objects of type `Item`.  There may be many routes under the `/item/:itemId` path, and each of these need to look up the item.  They may also share the same authorisation properties.  In this case, it may be useful to put this logic into an action builder.

First of all, we'll create a request object that adds an `Item`:

@[request-with-item](code/ScalaActionsComposition.scala)

Now we'll create an action builder that looks up that item when the request is made.  Note that this action builder is defined inside a method that takes the id of the item:

@[item-action-builder](code/ScalaActionsComposition.scala)

Now we can use this action builder for each item:

@[item-action-use](code/ScalaActionsComposition.scala)

## Authentication

One of the most common use cases for action composition is authentication.  We can easily implement our own authentication action builder like this:

@[authenticated-action-builder](code/ScalaActionsComposition.scala)

Play also provides a built in authentication action builder.  Information on this and how to use it can be found [here](api/scala/index.html#play.api.mvc.Security$$AuthenticatedBuilder$).

> **Note:** The built in authentication action builder is just a convenience helper to minimise the code necessary to implement authentication for simple cases, its implementation is very similar to the example above.
>
> If you have more complex requirements than can be met by the built in authentication action, then implementing your own is not only simple, it is recommended.

Play also provides a [[global filter API | ScalaHttpFilters]], which is useful for global cross cutting concerns.

> **Next:** [[Content negotiation | ScalaContentNegotiation]]
