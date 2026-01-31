---
title: "Craft GraphQL APIs in Elixir with Absinthe"
date: 2026-01-31
draft: false
tags:
  - book-highlights
---

# Part I — Build a GraphQL API

## Chapter 1: Meet GraphQL

- including relationships

When using a REST API, here are some questions we should probably ask ourselves:

- Can we get a user’s friends via /users/:id, or do we need to use a different resource URL?
- Are friends (or their names) included by default, or do we need to specify that we want them? How do we do that?
- When information about friends is returned, where will it be in the result?
- Is each friend’s data embedded, or just referenced, forcing us to do more requests to retrieve it? Will we need to paginate the friends list? How will this affect our performance on mobile devices, where bandwidth may be limited and additional requests have a negative effect on performance and battery life?

- one of the more important benefits that pro- viding one gives us automatically: user input validation
- GraphQL simplifies this dramatically by inverting the problem. With GraphQL you have a predefined schema with declared types and rules about the input that you can receive from users to retrieve data. When someone sends you data, it is checked by these rules before your code is executed.

- the relationships between objects are commonly referred to as “edges” (drawn as lines), and the objects themselves are called “nodes”
- All queries start at a root object we have on our schema graph (we’re calling it RootQueryType)
- One of the core value propositions of GraphQL is its ability to provide a common interface to different types of data store

## Chapter 2: Building a Schema
- GraphQL APIs are shaped by their schemas. In fact, it’s fair to say that because GraphQL schemas are more than just structural metadata and actually define behavior, they’re the API implementation itself.
- A GraphQL query is the way that API users can ask for specific pieces of information... we need to provide two things:
	- A way for users to request objects of the type  
	- A way for the system to retrieve (or resolve) the associated data
- Our :menu_items field doesn’t actually build the list of menu items yet. To do that, we have to retrieve the data for the field. GraphQL refers to this as res- olution, and it’s done by defining a resolver for our field
	- We’ve passed a function to the resolve macro to set the field’s resolver.
```
for menu_item <- PlateSlate.Repo.all(PlateSlate.Menu.Item) do   Map.get(menu_item, :name)
end
```
- use the "@desc" attribute to add descriptions that are displayed in the GrapgQL GUI
```
@desc "The list of available items on the menu" field :menu_items, list_of(:menu_item) do

«Menu item field definition»

end
```

## Chapter 3: Taking User Input
- GraphQL takes a more declarative approach, however, by defining input as part of your API schema and supporting type validations as a core feature.
- GraphQL documents are made up of fields. The user lists the fields they would like, and the schema uses its definition of those fields to resolve the pieces of data that match.
- ...GraphQL has the concept of field arguments: a way for users to provide input to fields that can be used to parameterize their queries.
```
arg :<some-key>, :string
```
- HTTP 400 response code from Absinthe. This indicates that one or more errors occurred that prevented query execution.
- GraphQL variables—a way to insert dynamic argument values provided alongside (rather than inside) the static GraphQL document.
```
query ($term: String) { 
  menuItems(matching: $term) {
    name 
  }
}
```
- Variable declarations are provided directly before the curly braces that start the body of an operation, and are placed inside a set of parentheses. Variable names start with a dollar sign ($), and their GraphQL types follow after a colon (:) and a space character. If we were declaring multiple variables, we’d list them separated by commas.
- A GraphQL enumeration (or enum, as it’s generally called) is a special type of scalar that has a defined, finite set of possible values. Enums are a good choice if the possible values are well defined and unlikely to change, if those values are short (one or maybe two words long), and if they’re not a pair of values that are more clearly represented by a boolean flag.
	- By convention, enum values are passed in all uppercase letters
- We can collect multiple arguments and model them as a special object type used just for argument values
- Let’s say, for instance, that we wanted to ensure that a filter was always provided to the :field field, regardless of what the document says should be mandatory. We can do this by using the Absinthe non_null macro, wrapping the argument type


## Chapter 4: Adding Flexibility
- Capturing and distilling an entire domain model into a GraphQL schema can be quite the undertaking, and it’s important to keep your schema as main- tainable and flexible as possible as it grows to describe the needs of your API
- You’ll discover how using abstract types like interfaces and unions can make your API simpler, and you’ll learn how users can leverage GraphQL fragments to keep their documents shorter and easier to understand
- Absinthe schemas are compiled
- To support breaking up a large object, Absinthe provides another macro, import_fields, that we can use.
	- Instead of defining the fields directly in the root query object type, we can pull them out and put them into separate types (which we can place in other type modules)
- We opted for extracting the types along the same lines as our Ecto schemas are organized
- The union macro is used to create our type, and it works a lot like object. The types macro, used inside the union scope, sets its types.
	- a union type means “one of these types.”
- There’s one concern that this neat new ability to generalize a field result brings up: if a field can return different results with different shapes, how can users effectively select the data they want in the query
	- To do this, we make use of an important GraphQL feature, fragments
	- The inline fragment targets a type (introduced with on)
- Fields that begin with __ are reserved by GraphQL to support features like introspection
- GraphQL interfaces are similar to unions, with one key difference: they add a requirement that any member types must define a set of included fields.
	- selecting fields that have been declared on the interface aren’t subject to the same type of restrictions as selecting fields on unions (which have no such mechanism).
	- If there are fields in common, interfaces allow users to write more simple, readable GraphQL
- Named fragments are just like inline fragments, but they’re reusable.


## Chapter 5: Making a Change with Mutations
- you can think of GraphQL mutations as roughly analogous to POST, PUT, and DELETE operations
	- Unlike REST, however, the responses from GraphQL mutations can be tailored, just as with GraphQL query operations
- To support mutation operations, we need to define a root mutation type
```
mutation do  
  field :create_menu_item, :menu_item do
    arg :input, non_null(:menu_item_input) 
    resolve &Resolvers.Menu.create_item/3
  end
end
```
- Most mutations that create something return it as the result of the field
	- It’s easy to forget that you can’t use object types for user input; instead, you need to create input object types for use in arguments.
	- There are also some technical differences between objects and input objects. Input object fields can only be valid input types, which excludes unions, interfaces, and objects. You also can’t form cycles with input objects, whereas cycles are permitted with objects.


## Chapter 6: Going Live with Subscriptions
- Setting up connections for specific data feeds and managing the communica- tion across them is, to say the least, awkward in a world of “resources” tightly coupled to HTTP verbs
- GraphQL, on the other hand, puts near real-time data at the same first-class level as queries or mutations with what are called subscription operations. It’s these subscriptions that allow users to request data updates—using the same semantics as any other GraphQL request—and manage the life cycle of the data feed.
- This Absinthe.Subscription supervisor starts a number of processes that will handle broadcasting results and hold on to subscription documents. You pass the supervisor your PlateSlateWeb.Endpoint module because you’re using the Phoenix endpoint as the underlying pubsub mechanism.
- Subscriptions push data in response to events or actions within your system, so it isn’t enough to think of subscriptions as a purely API-related concern.
- GraphiQL expects variable input to be given in JSON format
- Testing a socket, then, is just a matter of sending it the data we need to configure our subscription, triggering a mutation, and then waiting for sub- scription data to get pushed to the test process.
- This pattern of connecting mutation and subscription fields to one another is so common that Absinthe considers it a first-class concept and supports setting it as a trigger on subscription fields, avoiding the need to scatter publish/3 calls throughout your code base.
	- it’s best to use triggers when there’s a clear and sensible mapping of mutations to subscriptions because it helps place this information in a clear and central location. Placing the trigger topic functions next to the subscription topic function goes a long way toward keeping track of how each operation connects to the other.

# Part II: Publish Your API

## Chapter 7: Resolution Middleware
- a tool you can use to make resolvers shorter and more elegant by being able to reuse logic
- This Absinthe.Resolution middleware has been the driving force behind how our resolvers have operated this whole time, building the arguments to our resolvers, calling them, and then interpreting the result we’ve returned
- When you have specific fields on which you want to place middleware, you’ll want to reach for the middleware/2 macro
- Macro Approach
- Callback Approach
	- “All fields on the mutation object should run this middleware after resolution.”
- Middleware is an enormously important tool for keeping your resolvers focused, giving you an incredible amount of control over what happens when executing a document. It’s also a great feature for third-party packages that want to augment Absinthe’s resolution logic; the middleware/3 callback is a handy inte- gration point.

## Chapter 8: Securing with Authentication and Authorization

- The Absinthe feature that addresses this problem is called the execution context. It’s a place where we can set values that will be available to all of our resolvers.
	- the final argument passed to the resolver function is an Absinthe.Resolution struct, which includes the context
	- Importantly, the context is always available in every field at every level, and it’s this property that gives it its name
- The core functionality of the plug is the call/2 callback, which takes a %Plug. Conn{} struct. Inside this struct is a private key, which is a place for libraries like Absinthe to place values for later use
- Subscriptions and authorization are all about topic design. Topics are extremely cheap and should be used readily to help scope published data to precisely the clients that should be able to see it
- The me pattern is an approach where fields that always depend on whoever is viewing the API are placed on some object representing the current user so that the document’s structural hierarchy makes that dependency clear


## Chapter 9: Tuning Resolution
- The pipeline is at the heart of how Absinthe works, and manipulating the parts that make up the pipeline—phases—is key to many of the advanced features that Absinthe offers
	- Phases are the real workhorses of Absinthe and have names like Phase.Document.Validation.NoFragmentCycles and Phase.Document.Arguments.Normalize
- Broadly speaking, Absinthe phases fall into three categories:
	- Preparation phases
	- Execution phases
	- Result building phases
- can run resolvers async
- can batch resolvers
- In time, people involved in larger projects were able to build some abstractions, helpers, and conventions around the Absinthe.Middleware.Batch plugin that did a good job of addressing these issues. That effort has been extracted into the project Dataloader, which, while a generic tool not tied to GraphQL, is the perfect fit for what we’re trying to accomplish here.
	- Dataloader is a small package that defines a minimalist API for getting data in batches.
	- Similar to batch, on_load hands off control to the Absinthe.Middleware.Dataloader module


# Part III: Use Your API

## Chapter 10: Driving Phoenix Actions with GraphQL
- A directive is a type that’s defined in our schema, just like an object or a scalar, and we can use these types to annotate parts of our GraphQL docu- ments for special handling

```
import_types Absinthe.Phoenix.Types

# in the controller
@graphql """
  query Index @action(mode: INTERNAL) {
    menu_items
  }
"""
```

- With this in place, we have access to the :action directive, which we can use to annotate GraphQL queries in our controller
- The real value of the @action directive is that we can treat our GraphQL API as a first-class Elixir data source
	- The use of @put in our document indicates to Absinthe.Phoenix that instead of narrowing down the results of the menu_items field to only the fields in the selection set, we want to put those values into the previous result.

## Chapter 11: Integrating with the Frontend
- Since we’re doing a query operation to get our menu items, we could be using GET, but it’s easier and more consistent to encode our GraphQL document as part of an application/json POST body
- cors_plug
- there are two major client-side JavaScript frameworks that developers use to build user interfaces on top of GraphQL: Relay and the Apollo GraphQL platform
- I didn't read the Apollo/Relay bits