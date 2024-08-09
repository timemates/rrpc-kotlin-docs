# Instances

We provide mechanism to pass data through the chain of Request Interceptors, RPC and Response Interceptors. It's used
to provide additional data linked to the request (for example, request id). You can provide whatever you want!

## Defining the instance

All 'instances' have to implement the following interface:

```Kotlin
public interface ProvidableInstance {
    public val key: Key<*>

    public interface Key<TInstance : ProvidableInstance>
}
```

Here's an example of instance from library:

```Kotlin
@OptIn(ExperimentalSerializationApi::class)
@JvmInline
public value class ProtobufInstance(
    public val protobuf: ProtoBuf
) : ProvidableInstance {

    public companion object Key : 
        ProvidableInstance.Key<ProtobufInstance>

    override val key: ProvidableInstance.Key<ProtobufInstance>
        get() = Key
}
```

As you see, it implements `ProvidableInstance` what makes it able to be used in `InstanceContainer` that stores
instances through all the chain.

To provide your instance, just simply add it into your client / server configuration:
<tabs>
    <tab title="Server">
        <code-block lang="kotlin">
    val module = RSPModule { // this: RSPModuleBuilder
      // services and other ...
      instances {
        register(MyInstance())
      }
    }
</code-block>
    </tab>
    <tab title="Client">
        <code-block lang="kotlin">
    val configuration = RSPClientConfig {
      instances {
        register(MyInstance())
      }
    }
        </code-block>
    </tab>
</tabs>

Or if you need to provide it dynamically, you can provide it through `InterceptorContext`:

```Kotlin
class MyInterceptor : RequestInterceptor {
    override fun intercept(context: InterceptorContext<...>) {
        // some logic
        return context.modify {
            addLocalInstance(MyInstance())
        }
    }
}
```

Modified `InstanceContainer` is passed down through RPC (if server) and then Response Interceptors. So, it's stored
until request is finished. Here's the scheme:
<img src="interceptors_mental_model.svg" dark-src="interceptors_mental_model_dark.svg" width="450"  alt="Interceptors mental model"/>

## Retrieving instances

Instances can be retrieved in interceptors and inside RPC implementation on server-side (using `RequestContext`). For an
example, let's take interceptors case and serialize the metadata into proto, and print it:

```Kotlin
class MyInterceptor : RequestInterceptor {
    override fun intercept(context: InterceptorContext<...>) {
        // some logic
        context.instances
            .getInstance(ProtoBufInstance)
            .protobuf
            .encodeToByteArray(context.metadata)
            .let(::println)
            
        return context
    }
}
```
And inside RPC:
```Kotlin
suspend fun foo(context: RequestContext, data: Bar): FooBar {
    // the same
    context.instances
            .getInstance(ProtoBufInstance)
            .protobuf
            .encodeToByteArray(context.metadata)
            .let(::println)
            
    return // ...
}
```
