# Getting started

So you decided to go with rRPC, let's make it quick. What do you need?

<warning>
rRPC is still under development, backward compatibility for communication model is not guaranteed until 1.0. For now,
existing API is more likely to be as-is, but we need more feedback.

The documentation is not up to date for now, moreover, it's all about the upcoming release of
0.7.0 that is not released just yet.
</warning>

## Implementation

First of all, you need a gradle plugin and the following dependencies:

```Kotlin
plugins {
   id("org.timemates.rrpc") version ("%lib-version%")
}

// ...

dependencies {
    // for server (jvm, js, native)
    commonMainImplementation("org.timemates.rrpc:server-core:$version")
    
    // for client (jvm, js, native)
    commonMainImplementation("org.timemates.rrpc:client-core:$version")

    // for server & client (jvm, js, native)
    commonMainImplementation("org.timemates.rrpc:common-core:$version")
}
```

The supported platforms are the same as
for [`rsocket-kotlin`](https://github.com/rsocket/rsocket-kotlin?tab=readme-ov-file#supported-platforms-and-transports-)

## Initialization

rRPC generates code depending on the schema you specify in `.proto` files,
similar to gRPC. You can learn more about protobuf [here](https://protobuf.dev/).

### Gradle Plugin

Let's start with configuring rRPC gradle plugin:

```Kotlin
rrpc {
   // you might need to specify your main 
   // source set (if its name is not commonMain / main)
   targetSourceSet = "commonMain"
   
   // folder with .proto schema files (it may have multiple sources)
   protoSources.add("src/commonMain/proto")
   
   // global options
   options {
    permitPackageCycles = true
   }
      
   configurations {
    kotlin {
        // kotlin-specific options
        options {
            generateServer = true
            generateClient = true
      
            // folder where generated code will be put
            generationOutput = "generated/rrpc/src/commonMain"
        }
    }
   
    // or other languages (configurations)
    typescript {/* ... */}
    getByName("php") {/* ... */}
   }
}
```

Then, write your `.proto` files with services and types you need. For example:

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
    Unfortunately, we support only proto3 syntax.
</note>
After you finished with your ProtoBuf definitions, you can use the following:

```Bash
./gradlew :rrpc:generateCode
```

### Server

To implement your RPC on server side, simply implement abstract class
with the same name as in your ProtoBuf definition:

```Kotlin
class BarServiceImpl : BarService {
    override suspend fun buzz(context: RequestContext, data: Foo): Foo {
        delay(1000L)
        return Foo {
            test = "some text"
        }
    }
}
```

And then you can use it whether with WebSockets or Sockets (via Ktor):
<tabs>
    <tab title="Ktor WebSockets">
        <code-block lang="kotlin">
    fun Application.configureServer() {
        routing {
            rrpcEndpoint("/rrpc") { // this: rRPCModuleBuilder
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
    server.handlerJob.join() //wait for server to finish
        </code-block>
    </tab>
</tabs>

### Client
For client, you simply initialize the class with the name of service + 'Client':
```Kotlin
val configuration = rRPCClientConfig {
    // assume we have a rsocket client instance
    // for obtaining a right one, please refer to the
    // rsocket-kotlin documentation.
    rsocket = TODO()
    // ...
}

val barClient = BarServiceClient(configuration)
println(barClient.buzz().test)
```
All other things, like ability to reconnect automatically, you can find
at [the official documentation](https://github.com/rsocket/rsocket-kotlin/blob/master/README.md).

<seealso>
            <a href="" type="start" summary="Install and start quickly with rRPC">Getting started</a>
            <a href="" type="idea" summary="The idea and motivation behind rRPC development">Why rRPC?</a>
</seealso>
