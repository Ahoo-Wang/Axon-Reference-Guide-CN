---
description: Query Handlers
---

# 查询处理程序

The handling of a query comes down to an annotated handler returning the query's response. The goal of this chapter is to describe what such an `@QueryHandler` annotated method looks like, as well as describing the call order and response type options. For configuration of query handlers and the `QueryBus`, it is recommended to read the [Configuration](configuration.md) section.

## Writing a Query Handler

In Axon, an object may declare a number of query handler methods, by annotating them with the `@QueryHandler` annotation. The object in question is what you would refer to as the Query Handler, or Query Handling Component. For a query handler method, the first declared parameter defines which query message object it will receive.

Taking the 'Gift Card' domain which contains a `CardSummary` Query Model, we can assume there is a query message to fetch a single `CardSummary` instance. Let us define the format of the query message as follows:

```java
public class FetchCardSummaryQuery {

    private final String cardSummaryId;

    public FetchCardSummaryQuery(String cardSummaryId) {
        this.cardSummaryId = cardSummaryId;
    }
    // omitted getters, equals/hashCode, toString functions
}
```

As shown, we have a regular POJO that will fetch a `CardSummary` based on the `cardSummaryId` field. This `FetchCardSummaryQuery` will be [dispatched](query-dispatchers.md) to a handler that defines the given message as its first declared parameter. The handler will likely be contained in an object which is in charge of or has access to the `CardSummary` model in question:

```java
import org.axonframework.queryhandling.QueryHandler;

public class CardSummaryProjection {

    private Map<String, CardSummary> cardSummaryStorage;

    @QueryHandler // 1.
    public CardSummary handle(FetchCardSummaryQuery query) { // 2.
        return cardSummaryStorage.get(query.getCardSummaryId());
    }
    // omitted CardSummary event handlers which update the model
}
```

From the above sample we want to highlight two specifics when it comes to writing a query handler:

1. The `@QueryHandler` annotation which marks a function as a query handler method.
2. The method in question is defined by the return type `CardSummary`, which is called the query response type, and the `FetchCardSummaryQuery` which is the query payload.

> **Storing a Query Model**
>
> For the purpose of the example we have chosen to use a regular `Map` as the storage approach. In a real life system, this would be replaced by a form of database or repository layer for example.

## Query Handler Call Order

In all circumstances, at most one query handler method is invoked per query handling instance. Axon will search for the most specific method to invoke, using following rules:

1. On the actual instance level of the class hierarchy (as returned by `this.getClass()`), all annotated methods are evaluated
2. If one or more methods are found of which all parameters can be resolved to a value, the method with the most specific type is chosen and invoked
3. If no methods are found on this level of the class hierarchy, the super type is evaluated the same way
4. When the top level of the hierarchy is reached, and no suitable query handler is found, this query handling instance is ignored.

Note that similar to command handling, and unlike event handling, query handling does not take the class hierarchy of the query message into account.

```java
// assume QueryB extends QueryA 
// and    QueryC extends QueryB
// and    a single instance of SubHandler is registered

public class QueryHandler {

    @QueryHandler
    public MyResult handle(QueryA query) {
        // Return result
    }

    @QueryHandler
    public MyResult handle(QueryB query) {
        // Return result
    }

    @QueryHandler
    public MyResult handle(QueryC query) {
        // Return result

    }
}

public class SubQueryHandler extends QueryHandler {

    @QueryHandler
    public MyResult handleEx(QueryB query) {
        // Return result
    }
}
```

In the example above, the handler method of `SubQueryHandler` will be invoked for queries for `QueryB` and result `MyResult` the handler methods of `QueryHandler` are invoked for queries for `QueryA` and `QueryC` and result `MyResult`.

## Query Handler Return Values

Axon allows a multitude of return types for a query handler method, as defined [earlier](query-handlers.md#writing-a-query-handler) on this page. You should think of single objects and collections of objects, taking into account wildcards or generics too. Below we share a list of all the options which are supported and tested in the framework.

For clarity we make a deviation between single instance and multiple instances of a response type. This follows the requirement to specify the `ResponseType` when [dispatching a query](query-dispatchers.md), which expects the user to state if either a single result or multiple results are desired. Axon will use this `ResponseType` object to match a query with a query handler method, along side the query payload and query name.

### Supported Single Instance Return Values

To query for a single object, the `ReponseTypes#instanceOf(Class)` method should be used to create the required `ResponseType` object. This "instance-of-`Class`" `ResponseType` object in turn supports the following query handler return values:

* An exact match of `Class`
* A subtype of `Class`
* A generic bound to `Class`
* A `Future` of `Class`
* A primitive of `Class`
* An `Optional` of `Class`

> **Primitive Return Types**
>
> Among the usual Objects, it is also possible for queries to return primitive data types:
>
> ```java
> public class QueryHandler {
>  
>      @QueryHandler
>      public float handle(QueryA query) {
>      }
>  }
> ```
>
> Note that the querying party will retrieve a boxed result instead of the primitive type.

### Supported Multiple Instances Return Values

To query for a multiple objects, the `ReponseTypes#multipleInstancesOf(Class)` method should be used to create the required `ResponseType` object. This "multiple-instances-of-`Class`" `ResponseType` object in turn supports the following query handler return values:

* An array containing:
  * `Class`
  * A subtype of `Class`
  * A generic bound to `Class`
* An `Iterable` or a custom implementation of `Iterable` containing:
  * `Class`
  * A subtype `Class`
  * A generic bound to `Class`
  * A wildcard bound to `Class`
* A `Stream` of `Class`
* A `Future` of an `Iterable` of `Class`

### Unsupported Return Values

The following list contains method return values which are not supported when queried for:

* An array of primitive types
* A `Map` of a given key and value type
