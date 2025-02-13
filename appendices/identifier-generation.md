---
description: D. Identifier Generation
---

# D. 标识符生成

The Axon Framework uses an `IdentifierFactory` to generate all identifiers, whether they are for events, commands or queries. The default `IdentifierFactory` uses randomly generated `java.util.UUID` based identifiers. Although they are very safe to use, the process to generate them does not excel in performance.

`IdentifierFactory` is an abstract factory that uses Java's `ServiceLoader` (since Java 6) mechanism to find the implementation to use. This means you can create your own implementation of the factory and put the name of the implementation in a file called `/META-INF/services/org.axonframework.common.IdentifierFactory`. Java's `ServiceLoader` mechanism will detect that file and attempt to create an instance of the class named inside.

There are a few requirements for the `IdentifierFactory`. The implementation must:

1. Have its fully qualified class name as the contents of the `/META-INF/services/org.axonframework.common.IdentifierFactory` file on the classpath,
2. have an accessible zero-argument constructor,
3. extend `IdentifierFactory`,
4. be accessible by the context classloader of the application or by the classloader that loaded the `IdentifierFactory` class, and must
5. Be thread-safe.
