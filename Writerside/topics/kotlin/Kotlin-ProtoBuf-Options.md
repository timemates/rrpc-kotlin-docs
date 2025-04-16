# ProtoBuf Options Support

The rRPC code-generation supports ProtoBuf options on service and method levels.

You can retrieve options in either `InterceptorContext` (for both server and client) or `RequestContext` (server-only).
Options are considered as extra-information that shouldn't be correlated with main data.

## Server

For server, options are generated inside `descriptor` property. Here's an example:

```Kotlin
  override val descriptor: ServiceDescriptor = ServiceDescriptor(
    name = "TestService",
    procedures = listOf(
        ProcedureDescriptor.RequestResponse(
            name = "TestMethod",
            inputSerializer = TestMessage.serializer(),
            outputSerializer = TestMessage.serializer(),
            procedure = { context, data -> testMethod(context, data) },
            options = OptionsWithValue(
                // options are scoped to the messages namespace if defined
                // within them, you can override this behavior via builtin 
                // timemate.rrpc.extension_generation_strategy option
                TestMessage.customMethodOption to TestMethodOpt.TESTMETHODOPT_BETA,
            ),
        ),
    ),
    options = OptionsWithValue(
        ServiceOption.customServiceOption to "ServiceOptionValue",
    ),
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
        if (context.options[RPCOption.isAuthorizationRequired] == true)
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

