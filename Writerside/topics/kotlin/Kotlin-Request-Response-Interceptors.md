# Request/Response Interceptors

In the rRPC framework, interceptors are a powerful mechanism that allows you to modify or augment the behavior of request
processing in a modular and reusable manner. Interceptors can be applied both to requests (input) and responses (output).
They operate on the `InterceptorContext`, which carries data, metadata, options, and instances related to the
request or response.

## Request Interceptors

Request interceptors are executed before the actual handling of a request. They can be used to validate, log, modify, or
enrich the request data, metadata, and options before the request reaches its final destination.

<warning>
Interceptors shouldn't throw exceptions on its own, but rather pass it down by context. 
</warning>

### Simple Request Interceptor Example

A basic request interceptor might log the incoming data and continue the processing unchanged.

```Kotlin
@OptIn(ExperimentalInterceptorsApi::class)
class SimpleLoggingRequestInterceptor : RequestInterceptor {
    override fun intercept(
        context: InterceptorContext<ClientMetadata>
    ): InterceptorContext<ClientMetadata> {
        println("Received request: ${context.metadata.serviceName}#${context.metadata.procedureName}")
        return context // Continue with the unmodified context
    }
}
```

- `intercept(context: InterceptorContext<ClientMetadata>)`: This method is the entry point for the interceptor. It
  receives the current InterceptorContext.
- `return context`: Passes the unmodified context to the next interceptor in the chain or to the request handler if this
  is the last interceptor.

### Complex Request Interceptor Example

A more complex request interceptor might handle both single-value and streaming requests differently, logging each piece
of data as it passes through.

```Kotlin
@OptIn(ExperimentalInterceptorsApi::class)
class DetailedLoggingRequestInterceptor : RequestInterceptor {
    override fun intercept(
        context: InterceptorContext<ClientMetadata>
    ): InterceptorContext<ClientMetadata> {
        val reference = "${context.metadata.serviceName}#${context.metadata.procedureName}"

        when (context.data) {
            is DataVariant.Single -> {
                println("$reference(${context.data.requireSingle()})")
            }

            is DataVariant.Streaming -> {
                val id = UUID.randomUUID()
                return context.modify {
                    data = (context.data as DataVariant.Streaming).onEach {
                        println("streaming $id: $reference($it)")
                    }
                }
            }

            is DataVariant.Failure -> {
                println("Failed to call $reference:")
                (context.data as DataVariant.Failure).exception.printStackTrace()
            }
        }

        return context
    }
}
```

If interceptors chain returns `DataVariant.Failure`, it will throw an exception and process it to the response
interceptors, and then to the client.

## Response Interceptors
Response interceptors are executed after the request has been processed but before the response is sent back to the
client. They can be used to modify the response data, log results, or enforce additional security checks.

### Example
A basic response interceptor might simply log the outgoing response data.
```Kotlin
@OptIn(ExperimentalInterceptorsApi::class)
public class SimpleLoggingResponseInterceptor : ResponseInterceptor {
    override fun intercept(
        context: InterceptorContext<ServerMetadata>
    ): InterceptorContext<ServerMetadata> {
        println("Sending response: ${context.metadata.serviceName}#${context.metadata.procedureName}")
        return context // Continue with the unmodified context
    }
}
```

## Registering interceptors
To register your interceptor, just add it to your rRPCClientConfig / rRPCModule:
<warning>
  The order of interceptors is important – it's applied strictly by how they're added. If you have some kind of
  exception handler or logging mechanism, ensure that you have it as a last one.
</warning>

<tabs>
    <tab title="Server">
        <code-block lang="kotlin">
    val module = RRpcModule { // this: RRpcModuleBuilder
      // services and other ...
      interceptors {
        request(MyRequestInterceptor())
        response(MyRequestInterceptor())
      }
    }
</code-block>
    </tab>
    <tab title="Client">
        <code-block lang="kotlin">
    val configuration = RRpcClientConfig.create {
      interceptors {
        request(MyRequestInterceptor())
        response(MyRequestInterceptor())
      }
    }
        </code-block>
    </tab>
</tabs>


