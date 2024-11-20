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
    targetSourceSet.set("commonMain") // Specify the source set for generated code
    protosInput.set(listOf("src/main/proto")) // Define where to find .proto files

    kotlin {
        output = "build/generated/kotlin" // Specify output directory
        clientGeneration = true // Generate client stubs
        serverGeneration = true // Generate server stubs
        typeGeneration = true // Generate data types
        metadataEnabled = true // Enable metadata generation
        metadataScopeName = "MyScope" // Define metadata scope
    }
}
```

### Configuration Properties

#### `targetSourceSet`
- **Purpose:** Specifies the source set for code generation.
- **Why it's important:** Code generation tasks depend on this source set to correctly manage input and output directories for the build process. For instance, setting `commonMain` ensures that the generated files are included in the correct build phase. Especially useful for the custom source-set layouts.
- **Default:** `null` (falls back to default source sets like `commonMain` or `main`).

```Kotlin
targetSourceSet.set("commonMain")
```
#### `protosInput`
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