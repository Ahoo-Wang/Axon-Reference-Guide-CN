---
description: Sagas
---

# Sagas

Similar to command handling components, sagas have a clearly defined interface: they only respond to events. On the other hand, sagas often have a notion of time and may interact with other components as part of their event handling process. Axon Framework's test support module contains fixtures that help you write tests for sagas.

Each test fixture contains three phases, similar to those of the command handling component fixture described in the previous section.

* Given certain events (from certain aggregates),
* when an event arrives or time elapses,
* expect certain behavior or state.

Both the "given" and the "when" phases accept events as part of their interaction. During the "given" phase, all side effects, such as generated commands are ignored, when possible. During the "when" phase, on the other hand, events and commands generated from the saga are recorded and can be verified.

The following code sample shows an example of how the fixtures can be used to test a saga that sends a notification if an invoice is not paid within 30 days:

```java
FixtureConfiguration<InvoicingSaga> fixture = new SagaTestFixture<>(InvoicingSaga.class);
fixture.givenAggregate(invoiceId).published(new InvoiceCreatedEvent()) 
       .whenTimeElapses(Duration.ofDays(31)) 
       .expectDispatchedCommandsMatching(Matchers.listWithAllOf(aMarkAsOverdueCommand())); 
       // or, to match against the payload of a Command Message only 
       .expectDispatchedCommandsMatching(Matchers.payloadsMatching(Matchers.listWithAllOf(aMarkAsOverdueCommand())));
```

Sagas can dispatch commands using a callback to be notified of command processing results. Since there is no actual command handling done in tests, the behavior is defined using a `CallbackBehavior` object. This object is registered using `setCallbackBehavior()` on the fixture and defines if and how the callback must be invoked when a command is dispatched.

Instead of using a `CommandBus` directly, you can also use command gateways. See below on how to specify their behavior.

Often, sagas will interact with resources. These resources aren't part of the saga its state, but are injected after a saga is loaded or created. The test fixtures allow you to register resources that need to be injected in the saga. To register a resource, simply invoke the `fixture.registerResource(Object)` method with the resource as parameter. The fixture will detect appropriate setter methods or fields (annotated with `@Inject`) on the saga and invoke it with an available resource.

> **Tip**
>
> It can be very useful to inject mock objects (e.g. Mockito or Easymock) into your Saga. It allows you to verify that the saga interacts correctly with your external resources.

Command gateways provide sagas with an easier way to dispatch Commands. Using a custom command gateway also makes it easier to create a mock or stub to define its behavior in tests. When providing a mock or stub, however, the actual command might not be dispatched, making it impossible to verify the sent commands in the test fixture.

Therefore, the fixture provides two methods that allow you to register Command Gateways and optionally a mock defining its behavior: `registerCommandGateway(Class)` and `registerCommandGateway(Class, Object)`. Both methods return an instance of the given class that represents the gateway to use. This instance is also registered as a resource, to make it eligible for resource injection.

When the `registerCommandGateway(Class)` is used to register a gateway, it dispatches Commands to the CommandBus managed by the fixture. The behavior of the gateway is mostly defined by the `CallbackBehavior` defined on the fixture. If no explicit `CallbackBehavior` is provided, callbacks are not invoked, making it impossible to provide any return value for the gateway.

When the `registerCommandGateway(Class, Object)` is used to register a gateway, the second parameter is used to define the behavior of the gateway.

The test fixture tries to eliminate elapsing system time where possible. This means that it will appear that no time elapses while the test executes, unless you explicitly state so using `whenTimeElapses()`. All events will have the timestamp of the moment the test fixture was created.

Having the time stopped during the test makes it easier to predict at what time events are scheduled for publication. If your test case verifies that an event is scheduled for publication in 30 seconds, it will remain 30 seconds, regardless of the time taken between actual scheduling and test execution.

> **Note**
>
> The fixture uses a `StubScheduler` for time based activity, such as scheduling events and advancing time. Fixtures will set the timestamp of any events sent to the Saga instance to the time of this scheduler. This means time is 'stopped' as soon as the fixture starts, and may be advanced deterministically using the `whenTimeAdvanceTo` and `whenTimeElapses` methods.

You can also use the `StubEventScheduler` independently of the test fixtures if you need to test scheduling of events. This `EventScheduler` implementation allows you to verify which events are scheduled for which time and gives you options to manipulate the progress of time. You can either advance time with a specific `Duration`, move the clock to a specific date and time or advance time to the next scheduled event. All these operations will return the events scheduled within the progressed interval.
