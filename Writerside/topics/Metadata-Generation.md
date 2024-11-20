# Schema Metadata
## Overview
Schema metadata generation is a feature of rRPC’s code-generation tools, available in both the Gradle Plugin and CLI. 
It generates a static representation of the API schema during build time, avoiding runtime reflection. 
This metadata is primarily used for debugging and introspection purposes, especially in conjunction with tools like the
**rRPC Inspector app**.

## How It Works
When enabled, the code generator creates Kotlin metadata files that provide comprehensive information about the API schema, including:
- **Services** and their methods.
- **Request/Response types** for each method.
- **File-level and declaration-level options** from `.proto` files.

The metadata is generated as part of the build process, ensuring no runtime overhead. In addition, it has as much
 information as any generator at the generation time.

## Enabling Metadata Generation

### Using Gradle Plugin
To enable metadata generation, configure the `rrpc` block in your `build.gradle.kts` file:
```kotlin
rrpc {
    kotlin {
        metadataEnabled = true // Enable metadata generation
         // Optional: Define a metadata scope 
         // if not specified – global instance used instead
        metadataScopeName = "MyCustomScope"
    }
}
```

### Using CLI
For projects that don’t use Gradle, enable metadata generation via CLI:
```Bash
rrgcli \
  --protos_input=src/main/proto \
  --kotlin_output=build/generated \
  --kotlin_metadata_generation=true \
  --kotlin_metadata_scope_name=MyCustomScope
```

## Use Cases

### Debugging with rRPC Inspector
The metadata integrates seamlessly with **rRPC Inspector**, providing a visual representation of the API schema. This helps developers:
- Debug API services and methods.
- Verify .proto definitions.
### Runtime Schema Analysis
Although the metadata is static, it can be used at runtime for tasks like:
- Validating incoming requests.
- Custom routing or method discovery.
### API Documentation
Metadata can be consumed to generate automated documentation, reflecting the latest .proto schema.

## Best Practices
1.	**Enable Metadata Only When Necessary**
Use this feature when debugging, generating documentation, or using rRPC Inspector.
2.	**Use Scoped Metadata**
Organize metadata into logical groupings using metadataScopeName, especially for large or multi-team projects.
3.	**Automate in CI/CD Pipelines**
Integrate metadata generation into build pipelines to ensure schema changes are reflected consistently.