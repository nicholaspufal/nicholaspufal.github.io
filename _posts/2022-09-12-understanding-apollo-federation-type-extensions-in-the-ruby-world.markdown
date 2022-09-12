---
layout: post
title: "Understanding Apollo Federation Type Extensions in the Ruby World"
date: 2022-09-12 09:00
comments: true
---

----

_Cross-post: original posted on the [Doximity Technology Blog](https://technology.doximity.com/articles/understanding-apollo-federation-type-extensions-in-the-ruby-world)_

Type extensions in Apollo Federation are perhaps one of the most confusing concepts to translate into code. This article covers the importance of type extensions in a distributed graph as well as how to implement them properly using the [Apollo Federation Ruby implementation](https://github.com/Gusto/apollo-federation-ruby) (NOTE: code snippets below use Federation v1 and `apollo-federation ~> 2.0`)

### Covering the basics

In Apollo Federation we have the concept of entities, which are nothing more than GraphQL types with special annotations to declare how to identify them across subgraphs uniquely. Think about them as primary keys in a database schema.

Such entities may be referenced from other subgraphs, and in order for that to work you need a type extension.

### Why should I care about a type extension?

The main selling point for Apollo Federation is the ability to build a distributed graph with the separation of concerns in mind. There are times, however, when an entity declared by one subgraph may need to be used by another subgraph.

Imagine an orders subgraph that declares an `Order` type and a users subgraph that declares a `User` type. You may want your `User` entity to return a list of orders made by the logged-in user but you don't want any orders knowledge to leak into the users domain. The way to do that is via a `User` type extension performed by the orders subgraph.

### What does a type extension look like?

Again, let's use users and orders subgraphs as an example here.

The users subgraph declares a `User` entity that looks like this:

```ruby
# code below lives in the users subgraph
class User < BaseObject
  key fields: :id

  field :id, ID, null: false
  field :username, String, null: false
  field :name, String, null: false
end
```

Because the type includes this special `key` annotation, Apollo Federation knowns 1) that field `id` represents its "primary key", and 2) that users is the authoritative subgraph for `User`.

In order to define the field `orders` for the type `User` we declare a `User` type extension **from the orders subgraph**:

```ruby
# code below lives in the orders subgraph
ORDERS = [
  {
    id: '1',
    userId: '1',
    totalAmount: 100.00
  },
  {
    id: '2',
    userId: '2',
    totalAmount: 20.00
  },
  {
    id: '3',
    userId: '1',
    totalAmount: 10.00
  }
].freeze

class Order < BaseObject
  field :id, ID, null: false
  field :total_amount, Float, null: false

  def total_amount
    object[:totalAmount]
  end
end

class User < BaseObject
  key fields: :id
  extend_type

  field :id, ID, null: false, external: true
  field :orders, [Order], null: true

  def orders
    ORDERS.find_all { |order| order[:userId] == object[:id] }
  end
end
```

As seen above, the orders subgraph has to include a few things 1) the `extend_type` special keyword to indicate this is a type extension, 2) a `key` that matches the one defined by the entity (`id` in the case of `User`), and 3) the special directive `external: true` that indicates that the field `id`, although being fed by this subgraph, is owned by another subgraph.

Finally, let's say that the Query top-level field is declared as follows from the users subgraph:

```ruby
# code below lives in the users subgraph
USERS = [
  { id: '1', name: 'Mr Foo', username: 'mr_foo', },
  { id: '2', name: 'Mr Bar', username: 'mr_bar', },
].freeze

class Query < BaseObject
  field :me, User, null: true

  def me
    # Imagine this would be based on the user's session instead
    USERS[0]
  end
end

class UserSchema < GraphQL::Schema
  include ApolloFederation::Schema

  query(Query)
end
```

So, how will Apollo Federation resolve a request like the following?
```graphql
{
  me {
    orders {
      id
    }
  }
}
```

The query planner's plan will look like the following:
1. Reach out to the users subgraph to resolve `me`
2. After executing step 1, call the orders subgraph to resolve the `orders` field. This will be a special [entities request ](https://representations-typecondition-fix--apollo-federation-docs.netlify.app/docs/federation/federation-spec/#query_entities).
3. Apollo Federation Ruby will attempt to resolve the reference in 2 mutually-exclusive ways:

a. if the GraphQL Ruby type declaration responds to the class method `resolve_reference` (singular) it will invoke that method multiple times, _once for each of the identifiers_. Each time the method will be called with two arguments: the first a hash that represents a single reference (e.g. `{"__typename"=>"User", "id"=>"1"}`), and the other the GraphQL context object.

b. if the GraphQL Ruby type declaration doesn't respond to any of the above, then it will just return the `reference` hash as is. This means that whenever you use the special keyword `object` from any of that type's resolvers, you will get this hash (e.g. `{:__typename=>"User", :id=>"1"}`). This is the reason why in my example above I'm using `object[:id]` to find all orders belonging to the current user's id.

At this point, you might be wondering: what's the point of implementing your own `resolve_reference`?

The example above was simple, with only one extra field (`orders`) added as part of the type extension and with `ORDERS` being an in-memory data store. To answer the question, let's imagine we now want to add a `latest_order` field to `User` as well:

```ruby
# code below lives in the orders subgraph
class User < BaseObject
  key fields: :id
  extend_type

  field :id, ID, null: false, external: true
  field :latest_order, Order, null: true
  field :orders, [Order], null: true

  def latest_order
    ORDERS.last
  end

  def orders
    ORDERS.find_all { |order| order[:userId] == object[:id] }
  end
end
```
Also let's imagine `ORDERS` is instead an ActiveRecord model, querying from a database, or some Ruby code performing an API call. Can you spot the problem here? You would be making one separate request/query per field. Introducing a custom references resolver would prevent that from happening:

```ruby
# code below lives in the orders subgraph
class User < BaseObject
  key fields: :id
  extend_type

  field :id, ID, null: false, external: true
  field :latest_order, Order, null: true
  field :orders, [Order], null: true

  def latest_order
    object.last
  end

  def orders
    object
  end

  def self.resolve_reference(reference, _context)
    ORDERS.find_all { |order| order[:userId] == reference[:id] }
  end
end
```

This is much better. From now on, Apollo Federation will first call `resolve_reference` before invoking any field resolvers. If this was a database or an API call, I could fetch all data needed only once and then use `object` from each field resolver to reference that result.

We can go a step further: say we had a use case here to return a list of orders for a collection of users instead. In the above example we would still experience N+1s because for each user reference `resolve_reference` would be invoked. To prevent that from happening we can make use of the [batch loader gem](https://github.com/exAspArk/batch-loader):

```ruby
# code below lives in the orders subgraph
class User < BaseObject
  key fields: :id
  extend_type

  field :id, ID, null: false, external: true
  field :latest_order, Order, null: true
  field :orders, [Order], null: true

  def latest_order
    object.last
  end

  def orders
    object
  end

  def self.resolve_reference(reference, _context)
    ::BatchLoader::GraphQL.for(reference[:id]).batch do |user_ids, loader|
      # bulk load orders for users
    end
  end
end
```

### Good to know
* If you are implementing a type extension in a subgraph, but there are no fields in that subgraph that make use of that type, then make sure to list it as an [orphan type](https://graphql-ruby.org/schema/definition#root-objects-introspection-and-orphan-types). Otherwise, it won't be part of any GraphQL schema dumps, and Apollo Federation won't "see" it.
* `resolve_reference` can be implemented in the type definition _and_ the type extension.
* Version `3.0` of `apollo-federation` introduced `resolve_references` (plural). So instead of using the batch loader gem in combination with `resolve_reference` (singular), like I did on my example, you could just implement this plural form. `resolve_references` would be called once for all references, and the first argument would then be an array of hashes, each hash representing a single reference (e.g. `{"__typename"=>"User", "id"=>"1"}`).

-----------

Special thanks goes out to James Hicks, Carlos Gabaldon and Bruno Miranda for reading drafts of this blog post, and for giving me their feedback!

Be sure to follow [@doximity_tech](https://twitter.com/doximity_tech) if you'd like to be notified about new blog posts.


