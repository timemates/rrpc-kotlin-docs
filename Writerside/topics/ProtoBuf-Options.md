# ProtoBuf Options Support

The RSP code-generation supports ProtoBuf options on service and method levels.

<note>
For now, option's retention is not supported. The same applies to the file-level options or
message options.
</note>

You can retrieve options in either `InterceptorContext` (for both server and client) or `RequestContext` (server-only).
Options are considered as extra-information that shouldn't be correlated with main data.

## Server

For server, options are generated inside `descriptor` property. Here's an example:

```Kotlin
public val descriptor: ServiceDescriptor =
    ServiceDescriptor(
        name = "TestService",
        procedures =
            listOf(
                ProcedureDescriptor.RequestResponse(
                    name = "test",
                    // ...
                    options =
                        Options(
                            RPCOption.testop to "example",
                            RPCOption.int_op to 42,
                            RPCOption.bool_op to true,
                            RPCOption.float_op to 3.14f,
                            RPCOption.double_op to 2.71828.toDouble(),
                            RPCOption.enum_op to CustomEnum.VALUE_ONE,
                            RPCOption.message_op to
                                CustomMessage.create {
                                    field1 = "hello"
                                    field2 = 0
                                },
                        ),
                ),
            ),
        options = Options(emptyMap())
    )

```
But, obviously, you probably won't access them directly, but inside interceptors
for some kind of logic (for example, authorization). So, there's an ability to retrieve
and actually modify the list of options that is specified for the method client is demanding to execute:
```Kotlin
@OptIn(ExperimentalInterceptorsApi::class)
public class AuthInterceptor : RequestInterceptor {
    override fun intercept(
        context: InterceptorContext<ClientMetadata>
    ): InterceptorContext<ServerMetadata> {
        if(context.options[RPCOption.isAuthorizationRequired] == true)
            // some auth logic
            
        return context // Continue with the unmodified context
    }
}
```
In addition, you can access it in your RPC method by using `context: RequestContext` parameter:
```Kotlin
suspend fun foo(context: RequestContext, data: Bar): FooBar {
    println(context.options)
    // ...
}
```

## Client
For clients, you can access options only through interceptors. It's done in a similar way as on the server.

