---
description: Implementations
---

# 实现

When it comes to dispatching queries, as explained in the [Dispatching Queries](query-dispatchers.md) section, there are a couple of implementations when it comes to actually sending the query message in question. The next sections provide an overview of the possible implementations, as well as pointing out how to set up query dispatching infrastructure with Axon.

## Query Gateway

The query gateway is a convenient interface towards the query dispatching mechanism. While you are not required to use a gateway to dispatch queries, it is generally the easiest option to do so.

Axon provides a `QueryGateway` interface and the `DefaultQueryGateway` implementation. The query gateway provides a number of methods that allow you to send a query and wait for a single or multiple results either synchronously, with a timeout or asynchronously. The query gateway needs to be configured with access to the query bus and a (possibly empty) list of `QueryDispatchInterceptor`s.

## Query Bus

The query bus is the mechanism that dispatches queries to query handlers. Queries are registered using the combination of the query request name and query response type. It is possible to register multiple handlers for the same request-response combination, which can be used to implement the scatter-gather pattern. When dispatching queries, the client must indicate whether it wants a response from a single handler or from all handlers.

### AxonServerQueryBus

Axon provides a query bus out of the box, the `AxonServerQueryBus`. It connects to the [AxonIQ AxonServer Server](../../axon-server-introduction.md) to send and receive Queries.

`AxonServerQueryBus` is a 'distributed query bus'. It uses a [`SimpleQueryBus`](implementations.md) to handle incoming queries on different JVM's by default.

{% tabs %}
{% tab title="Axon Configuration API" %}
Declare dependencies:

```
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-server-connector</artifactId>
    <version>${axon.version}</version>
</dependency>
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-configuration</artifactId>
    <version>${axon.version}</version>
</dependency>
```

Configure your application:

```java
// Returns a Configurer instance with default components configured. 
// `AxonServerQueryBus` is configured as Query Bus by default.
Configurer configurer = DefaultConfigurer.defaultConfiguration();
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
By simply declaring dependency to `axon-spring-boot-starter`, Axon will automatically configure the Axon Server Query Bus:

```
<!--somewhere in the POM file-->
<dependency>
    <groupId>org.axonframework</groupId>
    <artifactId>axon-spring-boot-starter</artifactId>
    <version>${axon.version}</version>
</dependency>
```

> **Excluding the Axon Server Connector**
>
> If you exclude the `axon-server-connector` dependency you will fallback to 'non-axon-server' query bus option, the `SimpleQueryBus` (see [below](implementations.md))
{% endtab %}
{% endtabs %}

### SimpleQueryBus

The `SimpleQueryBus` does straightforward processing of queries in the thread that dispatches them. To configure a `SimpleQueryBus` (instead of an `AxonServerQueryBus`):

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
Configurer configurer = DefaultConfigurer.defaultConfiguration()
    .configureQueryBus(
        c -> SimpleQueryBus.builder()
            .transactionManager(c.getComponent(TransactionManager.class))
            .messageMonitor(c.messageMonitor(SimpleQueryBus.class, "queryBus"))
            .build()
    );
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```
@Bean
public SimpleQueryBus queryBus(AxonConfiguration axonConfiguration, TransactionManager transactionManager) {
    return SimpleQueryBus.builder()
                         .messageMonitor(axonConfiguration.messageMonitor(QueryBus.class, "queryBus"))
                         .transactionManager(transactionManager)
                         .errorHandler(axonConfiguration.getComponent(
                                 QueryInvocationErrorHandler.class,
                                 () -> LoggingQueryInvocationErrorHandler.builder().build()
                         ))
                         .queryUpdateEmitter(axonConfiguration.getComponent(QueryUpdateEmitter.class))
                         .build();
}
```
{% endtab %}
{% endtabs %}
