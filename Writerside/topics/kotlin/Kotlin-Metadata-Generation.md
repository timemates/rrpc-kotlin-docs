# Metadata Generation

## Overview

Schema metadata generation is a feature of rRPC’s code generation tools, available via both the Gradle Plugin and the
`rrgcli` CLI tool.  
It produces a static Kotlin representation of the `.proto` schema during build time—capturing what would otherwise
require complex reflection at runtime.

The generated metadata includes details about services, methods, and types, and is especially useful for advanced
tooling, runtime validation, or debugging workflows.
Includes all information that is available for plugins (shares the same schema).

## How It Works

When enabled, the Kotlin generator plugin outputs schema metadata alongside your usual client/server code or data
models.

This metadata contains:

- A complete list of **services** and **RPC methods**.
- Associated **request and response types**.
- Any **custom options** declared at the file, message, service, or method level.

This information is captured directly at code generation time, meaning it reflects exactly what the generator sees —
without any runtime overhead or dynamic parsing.

## Enabling Metadata Generation

Schema metadata can be enabled via either the Gradle plugin or the CLI.

### Using Gradle

You can enable metadata generation inside the `rrpc` DSL block by passing `metadata_generation` as a plugin option:

```kotlin
dependencies {
    // required minimum dependency to work with metadata
    implementation("app.timemate.rrpc:common-metadata:$generatorVersion")

    // implement only in case you need to query runtime the remote server for
    // schema (provides a convenient client to do it)
    implementation("app.timemate.rrpc:client-metadata")

    // implement only in case you need to expose your schema as a service
    implementation("app.timemate.rrpc:server-metadata")
}

rrpc {
    inputs {
        source {
            directory(project.file("src/main/proto"))
        }
    }

    outputFolder = layout.buildDirectory.file("generated/rrpc")

    plugins {
        add(
            notation = "app.timemate.rrpc:kotlin-generator:SNAPSHOT",
            type = PluginDependencyType.JAR,
        ) {
            option("metadata_generation", true)
            // optional, if not specified, global scope will be used
            // take into account that scoped modules don't have access 
            // to global scope.
            option("metadata_scope_name", "MyCustomScope")
        }
    }
}
```
<note>
For security measures, generated code does not contain <code>basePath</code> for any location field specified in the schema.
The only path that is retained is <code>relativePath</code> that must be more than enough
(<code>basePath</code> is usually used for generation-time errors).
</note>
If you omit `metadata_scope_name`, the metadata will be stored in a global container.

### Using CLI

When using the CLI, pass the following options to the Kotlin plugin via scoped flags:

```bash
rrgcli \
  --source_input=src/main/proto \
  --gen_output=build/generated \
  --plugin="java -jar ./generator.jar" \
  --rrpc-kotlin-gen:metadata_generation=true \
  --rrpc-kotlin-gen:metadata_scope_name=MyCustomScope
```

This will generate metadata that tools (or your app) can consume at runtime to understand and explore the service
layout.

## Use Cases

### Debugging with rRPC Inspector

If you integrate schema metadata with the **rRPC Inspector** (or other tooling),
you’ll be able to visually browse and inspect your entire API schema.

This includes:

- Listing all services and methods.
- Examining type structures.
- Reviewing custom `.proto` options.

<note>
The rRPC Inspector is under development. Not available for now.
</note>

### Runtime Schema Analysis

Although metadata is static, you can expose it through a service or utility class to:

- Perform **request validation** against declared method types.
- Create custom **routing** or **discovery logic**. In addition, if you have complex options logic
  based on message options, you may use metadata generation for such purposes.
- Enable **runtime self-description** for other tooling or SDKs.

### API Documentation

Tools can consume this metadata to generate documentation without parsing `.proto` files directly.  
This makes it possible to document your schema automatically as part of the build process.

<note>We're currently researching the way of possible support of API documentation. You may use protoc-based documentation generation for now.
Probably, the documentation will be available through rRPC Inspector.</note>

## Best Practices

1. **Enable Metadata Only When Needed**  
   Avoid including metadata in production builds unless required. It adds extra classes and may expose internal details.

2. **Use Scoped Metadata**  
   Specify a `metadata_scope_name` to group schemas logically. This is particularly helpful in multi-module or
   multi-team setups.

3. **Automate in CI/CD**  
   Add metadata generation to your CI to ensure schema consistency across tools, deployments, and environments.

### Using it runtime
Depending on your use-case, you may use it differently. Let's review how can we use it within our server flow.
Firstly, let's add our generated `SchemaMetadataModule` to instances:
```Kotlin
instances {
    // registers GlobalSchemaMetadataModule automatically
    metadataModules(FooSchemaMetadataModule)
    // alternative variant; take into account that you should register
    // GlobalSchemaMetadataModule explicitly in this case
    register(GlobalSchemaMetadataModule + FooSchemaMetadataModule())
}
```
<warning>
Take into account that scoped metadata module is not registered in global scope automatically. If you need such behaviour,
you must register it yourself via <code>GlobalSchemaMetadataModule.register(ScopedSchemaMetadataModule())</code>
</warning>

After it, as usual, you may get the `SchemaMetadataModule` via instances property inside context:

```Kotlin
@OptIn(ExperimentalInterceptorsApi::class)
public class DumbInterceptor : RequestRRpcInterceptor {
    override fun intercept(
        context: InterceptorContext<ClientMetadata>
    ): InterceptorContext<ClientMetadata> {
        // for simplicity of example
        // any generated message is subtype of [ProtoType]
        context.data as DataVariant.Single<ProtoType>
        
        val isDataDeprecated = context.instances.metadataModule
            .resolveMessage(context.data.definition.url)!!
            .options[RSOption.DEPRECATED]
        
        if (isDataDeprecated) {
            return context.modify {
                data(
                    IllegalArgumentException(
                        "The input message is deprecated"
                    )
                )
            }
        }
        
        return context // Continue with the unmodified context
    }
}
```
In the same way, you may access it within RPC and without RRpc context at all.
Mostly, it should be used to for pre-rending requirements, rather than using it directly.
We provide direct API (with caching) only to resolve files, types and extends.

<note>
Most likely, you can avoid it using already generated options for services and rpcs that is
available out-of-box.
</note>

### Exposing your schema
If you want to expose your schema via API and use it, for example, with rRPC Inspector, you may
use the `app.timemate.rrpc:server-metadata` artifact, as was mentioned before.
We have go-to solution that requires you only to register a dedicated service:
```Kotlin
val module = RRpcModule {
    services {
        // you may change what you want to expose by providing 
        // a specific module to it
        // in this way you may avoid exposing internal APIs
        register(DefaultSchemaMetadataService(FooBarSchemaMetadataModule()))
    }
}
```
If you need to secure this API, you may introduce an authorization interceptor, like this:
```kotlin
class SchemaMetadataAuthInterceptor : RequestRRpcInterceptor {
    override fun intercept(
        context: InterceptorContext<ClientMetadata>
    ): InterceptorContext<ClientMetadata> {
        
        if (context.metadata.[[[service|https://github.com/timemates/rrpc-kotlin/blob/267732dece8ab565754bb98109c6d535a558a70f/server/core/src/commonMain/kotlin/app/timemate/rrpc/server/ServicesContainer.kt#L29]]] is SchemaMetadataService) {
            val token = String(context.metadata.extra["access-token"])
            // ... auth logic
        }
        
        return context
    }
}
```
In addition, if you want to have fine-grained control over provided schema, you may
implement the `SchemaMetadataService` interface yourself:
```Kotlin
// SchemaMetadataService(...) mentioned before is just a 
// factory function with default implementation
class CustomSchemaMetadataService(
    vararg modules: SchemaMetadataModule
) : SchemaMetadataService() {
    override suspend fun resolveAllFiles(
        context: RequestContext, 
        data: ResolveAllFilesRequest,
    ): ResolveAllFilesRequest.Response {
        // your logic here
    }
    
    // other methods...
}
```

### Accessing exposed schema
To access exposed schema yourself rather than within tooling, you may use `app.timemate.rrpc:client-metadata` artifact.
```kotlin
val config = RRpcClientConfig.create {
    // ...
}

val schemaClient = SchemaMetadataServiceClient(config)
// As file count can be huge, page size should be specified. 
// Service on the other end by default restricts its number to 30 per
// chunk.
schemaClient.resolveAllFiles(
    pageSize = 30, 
    pageToken = null,
).toList()
```
You may refer to the documentation of `SchemaMetadataService` for more information.
