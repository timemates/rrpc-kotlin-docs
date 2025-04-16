# Getting Started

So you decided to go with rRPC — let's make it quick. What do you need?

<warning>
rRPC is still under development. Backward compatibility for the communication model is not guaranteed until version 1.0.  
The public API is becoming more stable, but further changes may occur based on user feedback.

Additionally, this documentation reflects the upcoming 0.7.0 release, which is not yet published.
</warning>

---

## Implementation

Start by applying the Gradle plugin and adding core dependencies:

```kotlin
plugins {
   id("app.timemate.rrpc") version("%lib-version%")
}

dependencies {
    // Server core library (JVM, JS, Native)
    commonMainImplementation("app.timemate.rrpc:server-core:$version")

    // Client core library (JVM, JS, Native)
    commonMainImplementation("app.timemate.rrpc:client-core:$version")

    // Shared logic or utilities
    commonMainImplementation("app.timemate.rrpc:common-core:$version")
}
```

Supported platforms
follow [rsocket-kotlin](https://github.com/rsocket/rsocket-kotlin?tab=readme-ov-file#supported-platforms-and-transports-).

## Code Generation Setup

rRPC uses `.proto` files to describe your API schema, similar to gRPC. You can learn more about `.proto` syntax and
semantics [here](https://protobuf.dev/).

### Gradle Plugin Configuration

Below is a working example using the **current API** of the rRPC Gradle plugin:

```kotlin
rrpc {
    inputs {
        source {
            directory(project.file("src/main/proto"))
        }

        // Optional: Use context dependencies for proto files that
        // shouldn't be included in default generation, unless used.
        context {
            artifact(libs.timemates.rrpc.common)
        }
    }

    // Allow proto package cycles, if needed
    permitPackageCycles = false

    // Where all generated sources will be placed
    outputFolder = layout.buildDirectory.file("generated/rrpc")

    plugins {
        add("app.timemate.rrpc:kotlin-generator:SNAPSHOT") {
            option("client_generation", true)
            option("server_generation", true)
        }
    }
}
```
> Learn more on [](Kotlin-Generation-Plugin.md) page which includes all possible options for generation and its quirks.

Once configured, place your `.proto` files into the source directory and define services and messages:

```
syntax = "proto3";

message Foo {
  string test = 1;
}

service BarService {
  rpc buzz(Foo) returns (Foo);
}
```

<note>
Only proto3 is supported. Generator will not warn of using 'proto2',
but its behavior will be according to the new standard. It means that, for example,
extending other than google.protobuf descriptors, will fail.
</note>

Then generate the Kotlin code with:

```bash
./gradlew generateRRpcCode
```

---

## Server Implementation

To implement the generated RPC service, simply create a class implementing the generated service interface:

```kotlin
class BarServiceImpl : BarService {
    override suspend fun buzz(context: RequestContext, data: Foo): Foo {
        delay(1000L)
        return Foo {
            test = "some text"
        }
    }
}
```

You can expose the service using Ktor or raw RSocket:

<tabs>
<tab title="Ktor WebSockets">
<code-block lang="kotlin">
fun Application.configureServer() {
    routing {
        rrpcEndpoint("/rrpc") {
            service(BarServiceImpl())
        }
    }
}
</code-block>
</tab>

<tab title="Ktor Raw Sockets">
<code-block lang="kotlin">
val server: TcpServer = server.bind(transport) {
    RSocketRequestHandler {
        rRPCModuleHandler(module).setup(this)
    }
}
server.handlerJob.join() // Wait for server to shut down
</code-block>
</tab>
</tabs>

---

## Client Usage

Clients are simple: initialize the client stub with configuration and you're ready to go:

```kotlin
val configuration = RRpcClientConfig.create {
    rsocket = TODO() // Provide RSocket instance
    // Optional config options
}

val barClient = BarServiceClient(configuration)
println(barClient.buzz(Foo { test = "hello" }).test)
```


<seealso>
            <a href="" type="start" summary="Install and start quickly with rRPC">Getting started</a>
            <a href="" type="idea" summary="The idea and motivation behind rRPC development">Why rRPC?</a>
</seealso>
