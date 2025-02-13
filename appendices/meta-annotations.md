---
description: C. Meta Annotations
---

# C. 元数据注解

Most annotations in Axon can be placed on other annotations as meta-annotations. When Axon scans for annotations, it will automatically scan meta-annotations as well. Annotations can override the properties defined on the meta-annotations, if desired.

For example, if you have a practice in your development team that payloads are always represented as JSON and you want the command name to be explicitly configured, you could create your own annotation:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.CONSTRUCTOR, ElementType.ANNOTATION_TYPE})
@CommandHandler(payloadType = JsonNode.class)
public @interface JsonCommandHandler {

    String commandName;

    String routingKey() default "";
}
```

By specifying the `payloadType` on the `@CommandHandler` meta-annotation, this becomes the value used for all Command Handlers annotated with `JsonCommandHandler`. These command handlers may (and should) still provide a parameter for the payload, but Axon will complain if it isn't a subclass of `JsonNode`.

The `commandName` attribute on the `JsonCommandHandler` annotation does not have a default value, and will therefore force developers to specify the name of the command. Note that to override values the attribute name must identical to the name on the `@CommandHandler` meta-annotation.

Lastly, the `routingKey` property is defined exactly as in the `@CommandHandler` annotation's specification to still allow developers to choose to provide a Routing Key when using the `JsonCommandHandler`.

When writing custom logic to access properties of annotations that may be meta-annotated, be sure to use the `AnnotationUtils#findAnnotationAttributes(AnnotatedElement, String)` method, or the `annotationAttributes` on the `MessageHandlingMember`. Using Java's annotation API will not take meta-annotations into consideration.
