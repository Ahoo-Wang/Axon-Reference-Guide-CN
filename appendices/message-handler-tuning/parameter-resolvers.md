---
description: Parameter Resolvers
---

# 参数解析器

You can configure additional `ParameterResolver`s by extending the `ParameterResolverFactory` class and creating a file named `/META-INF/services/org.axonframework.messaging.annotation.ParameterResolverFactory` containing the fully qualified name of the implementing class.

> **Caution**
>
> At this moment, OSGi support is limited due to the fact that the required headers are mentioned in the manifest file. The automatic detection of `ParameterResolverFactory` instances works in OSGi, but due to classloader limitations, it might be necessary to copy the contents of the `/META-INF/services/org.axonframework.messaging.annotation.ParameterResolverFactory` file to the OSGi bundle containing the classes to resolve parameters for (i.e. the event handler).
