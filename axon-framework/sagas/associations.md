---
description: Associations
---

# Associations

When a saga manages a transaction across multiple domain concepts, such as Order, Shipment, Invoice, etc, that saga needs to be associated with instances of those concepts. An association requires two parameters: the key, which identifies the type of association (Order, Shipment, etc) and a value, which represents the identifier of that concept.

Associating a saga with a concept is done in several ways. First of all, when a Saga is newly created when invoking a `@StartSaga` annotated event handler, it is automatically associated with the property identified in the `@SagaEventHandler` method. Any other association can be created using the `SagaLifecycle.associateWith(String key, String/Number value)` method. Use the `SagaLifecycle.removeAssociationWith(String key, String/Number value)` method to remove a specific association.

> Note
>
> The API to associate domain concepts within a Saga intentionally only allows a `String` or a `Number` as the identifying value, since a `String` representation of the identifier is required for the association value entry which is stored. Using simple identifier values in the API with a straightforward `String` representation is by design, as a `String` column entry in the database makes the comparison between database engines simpler. It is intentional that there is no `associateWith(String, Object)` for example, as the result of an `Object#toString()` call might provide unwieldy identifiers.

Imagine a saga that has been created for a transaction around an Order. The saga is automatically associated with the Order, as the method is annotated with `@StartSaga`. The saga is responsible for creating an Invoice for that Order, and tell Shipping to create a Shipment for it. Once both the Shipment has arrived and the Invoice has been paid, the transaction is completed and the saga is closed.

Here is the code for such a Saga:

{% tabs %}
{% tab title="Spring Boot AutoConfiguration" %}
```java
@Saga
public class OrderManagementSaga {

    private boolean paid = false;
    private boolean delivered = false;
    @Inject
    private transient CommandGateway commandGateway;

    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCreatedEvent event) {
        // client generated identifiers
        ShippingId shipmentId = createShipmentId();
        InvoiceId invoiceId = createInvoiceId();
        // associate the Saga with these values, before sending the commands
        SagaLifecycle.associateWith("shipmentId", shipmentId);
        SagaLifecycle.associateWith("invoiceId", invoiceId);
        // send the commands
        commandGateway.send(new PrepareShippingCommand(...));
        commandGateway.send(new CreateInvoiceCommand(...));
    }

    @SagaEventHandler(associationProperty = "shipmentId")
    public void handle(ShippingArrivedEvent event) {
        delivered = true;
        if (paid) { SagaLifecycle.end(); }
    }

    @SagaEventHandler(associationProperty = "invoiceId")
    public void handle(InvoicePaidEvent event) {
        paid = true;
        if (delivered) { SagaLifecycle.end(); }
    }

    // ...
}
```
{% endtab %}

{% tab title="Axon Configuration API" %}
```java
public class OrderManagementSaga {

    private boolean paid = false;
    private boolean delivered = false;
    @Inject
    private transient CommandGateway commandGateway;

    @StartSaga
    @SagaEventHandler(associationProperty = "orderId")
    public void handle(OrderCreatedEvent event) {
        // client generated identifiers
        ShippingId shipmentId = createShipmentId();
        InvoiceId invoiceId = createInvoiceId();
        // associate the Saga with these values, before sending the commands
        SagaLifecycle.associateWith("shipmentId", shipmentId);
        SagaLifecycle.associateWith("invoiceId", invoiceId);
        // send the commands
        commandGateway.send(new PrepareShippingCommand(...));
        commandGateway.send(new CreateInvoiceCommand(...));
    }

    @SagaEventHandler(associationProperty = "shipmentId")
    public void handle(ShippingArrivedEvent event) {
        delivered = true;
        if (paid) { SagaLifecycle.end(); }
    }

    @SagaEventHandler(associationProperty = "invoiceId")
    public void handle(InvoicePaidEvent event) {
        paid = true;
        if (delivered) { SagaLifecycle.end(); }
    }

    // ...
}
```
{% endtab %}
{% endtabs %}

By allowing clients to generate an identifier, a saga can be easily associated with a concept, without the need for a request-response type command. We associate the event with these concepts before publishing the command. This way, we are guaranteed to also catch events generated as part of this command. This will end the saga once the invoice is paid and the shipment has arrived.
