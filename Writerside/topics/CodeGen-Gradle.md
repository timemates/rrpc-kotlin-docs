# Gradle Plugin
The rRPC Gradle Plugin helps automate code generation for rRPC services in Gradle-based projects using `.proto` files.

<note>
If your project is not Gradle-based, you can use the <a href="CodeGen-CLI.md">rrgcli</a> for generating code.
</note>

## Applying the Plugin
Add the plugin to your `build.gradle.kts`:
```kotlin
plugins {
    id("org.timemates.rrpc") version "$latestVersion"
}
```

<note>
Make sure that you have Gradle Plugin Portal specified in your repositories.
</note>

## Configuring the Plugin
The plugin is configured using the rrpc extension, allowing customization for .proto file locations, output configurations, and generation behavior.

### Basic Example
```Kotlin
rrpc {
    // Configuring the input sources for .proto files
    inputs {
        // Adding a directory containing .proto files
        directory(file("src/main/proto"))

        // Adding an artifact (e.g., JAR or KLIB) containing .proto files
        artifact(
            file = file("libs/protos.jar"), 
            includes = listOf("org/timemates/internal")
        )

        // Adding an external dependency artifact
        artifact(
            id = "com.example:proto-library:1.0.0", 
            excludes = listOf("org/timemates/internal"),
        )
    }

    // Configuring plugins for code generation
    plugins {
        // enable kotlin generation
        kotlin {
            // Plugin-specific options
            output = "build/generated/rrpc-kotlin"
            clientGeneration = true
            serverGeneration = true
        }

        // External plugin configuration
        external("com.example:typescript-generator:2.0.0") {
            put("typescriptModule", "CommonJS")
            put("outputDir", "build/generated/typescript")
        }
    }

    // Enabling package cycles if necessary
    permitPackageCycles = true
}
```
The plugin automatically register its outputs for Kotlin (Multiplatform, Android, other named `main`) source sets for correct execution order. If you have
custom project, you can override its behavior by referencing to the outputs of `GenerateRRpcCodeTask`.

### Configuration Properties
#### `inputs`
- **Purpose**: Specifies the list of directories containing .proto files for generation.
- **Default**: An empty list.

#### Kotlin Configuration Options

These options are defined under the `kotlin` block:

- **`output`**: Specifies the directory where Kotlin code will be generated.
    - **Example**:
      `output = "build/generated/kotlin"`
- **`clientGeneration`**: Enables or disables generation of client stubs.
    - **Default**: `false`.
    - **Example**:
      `clientGeneration = true`
- **`serverGeneration`**: Enables or disables generation of server stubs.
    - **Default**: `false`.
    - **Example**:
      `serverGeneration = true`
- **`typeGeneration`**: Enables or disables generation of data types from `.proto` definitions.
    - **Default**: `false`.
    - **Example**:
      `typeGeneration = true`
- **`metadataEnabled`**: Enables generation of metadata. This can help in debugging (for rRPC Inspector) or providing additional context to the server about file-level or declaration-level options.
    - **Default**: `false`.
    - **Example**:
      `metadataEnabled = true`
- **`metadataScopeName`**: Specifies a scope for metadata, allowing you to filter or separate metadata from the global scope. Useful if you don’t want all metadata combined in a single container.
    - **Default**: Empty string (uses global scope).
    - **Example**:
      `metadataScopeName = "PotatoSchema"`

Learn more about Metadata Generation at the [separate page](Metadata-Generation.md).

## Additional Notes
- For projects with unusual directory structures or source sets, make sure to adjust targetSourceSet and protosInput accordingly.
- Generated files should not be modified manually. They are regenerated during each build.