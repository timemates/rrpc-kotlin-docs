# Custom plugins

Plugins are a powerful way to extend the functionality of a generator, enabling dynamic behavior, custom workflows or
generating code for different languages. By developing your own plugin, you can tailor the generator to suit specific
needs, such as processing custom inputs or integrating with external systems.

This guide will walk you through the steps of creating your own plugin, covering everything from adding dependencies to
handling specific signals and sending back appropriate responses. Whether you are building a plugin to add the behavior
to `rrgcli`, this tutorial will help you get started with the Plugin API.

## Step 1: Add Dependencies

Before starting development, you need to add the necessary dependencies for the plugin to interact with the host system
and exchange data. Ensure that your project includes the required libraries for communication via the Plugin API.

In your `build.gradle.kts` file (for Gradle projects), include the following dependencies:

```kotlin
dependencies {
    implementation("app.timemate.rrpc:generator-plugin-api:$latestVersion")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.0")
}
```
> 🔍 Replace $latestVersion with the actual version compatible with your setup.

Ensure you use the correct version numbers according to your setup.

## Step 2: Define the Adapter

### GenerationPluginService

The adapter is used to map the available options and process the input received from the generator. Its purpose
is to generate code, based on the input received:

```kotlin
public interface GenerationPluginService : PluginService {

    suspend fun generateCode(
        options: GenerationOptions,
        files: List<RSFile>,
    ): RequestStatusChange

    val options: List<OptionDescriptor>

    val name: String

    val description: String
}
```

For the reference, you might want to see the implementation
of [Kotlin Code Generator](https://github.com/timemates/rrpc-kotlin/tree/master/generator).

### ProcessorPluginService

If you need to modify or validate the incoming parsed input, you may want to use these types of plugins.

They have almost the same fields to be assigned by you, but instead of `generateCode` you will have `process`:

```kotlin
class DeprecationPluginService : PluginService {
    // ... assume all fields are defined
    suspend fun process(
        options: GenerationOptions,
        files: List<RSFile>,
        logger: RLogger,
    ): List<RSFile> {
        val resolver = RSResolver(files)
        return resolver.filter(
            object : RSEmptyVisitor<Unit, Boolean>() {
                override fun defaultHandler(
                    node: RSNode, 
                    data: Unit,
                ): Boolean {
                    return when (node) {
                        is RSField -> 
                            node.options[RSOption.DEPRECATED] != "true"
                        else -> false
                    }
                }
            }
        ).resolveAvailableFiles().toList()
    }
}
```

> The `generator-plugin-api` includes a convenient `filter` function that simplifies input filtering. It automatically
removes extension field usages if their definition is removed. However, this automatic cleanup only applies to extension
fields. If you need more fine-grained filtering, extend `RSVisitor` instead of `RSEmptyVisitor`. The traversal is automatic,
and you should return `false` for any node you want filtered out.

## Step 3: Enabling Plugin Service

The `PluginService` interface has a companion method to handle incoming signals from the
generator. All you need after implementing `PluginService`, is to do the following:

```kotlin
suspend fun main(args: Array<String>) {
    PluginService.main(
        args = args.asList(),
        input = System.`in`.source().buffer(),
        output = System.out.sink().buffer(),
        service = MyPluginService,
    )
}
```

## Step 4: Testing the Plugin

Currently, we don't provide dedicated testing tooling for plugins. However, if you're working within the JVM ecosystem,
you can write integration tests similar to those in `rrpc-kotlin:integration-tests`.

You may also write unit tests for your plugin logic to validate expected behavior in isolation. A simple way to test your
plugin end-to-end is by feeding mock `RSFile` inputs and checking the outputs/responses it produces.

Here is a minimal integration test pattern you can follow:

```kotlin
@Test
fun `plugin filters deprecated fields`() = runTest {
    val plugin = DeprecationPluginService()
    val result = plugin.process(options, mockRsFiles, mockLogger)
    // assertions here
}
```

## Step 5: Using the plugin

To use the plugin, simply specify the `--plugin` option when calling `rrgcli`:

```bash
rrgcli --plugin=/path/to/plugin --help
# or
rrgcli --plugin="java -jar plugin.jar" --help
```

> If you want to use your plugin inside a Gradle plugin, you may publish it either as a native binary with OS-specific classifiers or as a regular JAR with the main class specified.
> - For Windows: plugin-name-windows-x86_64.exe
> - For Linux: plugin-name-linux-x86_64 (without extension)
> - For MacOS: plugin-name-macos-aarch64 (without extension)

---

For detailed source-level documentation, refer to the source code kdoc. This guide focuses on the conceptual steps and tips to help you implement, test, and integrate your plugin efficiently.