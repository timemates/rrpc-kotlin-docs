# Developing your own plugins
Plugins are a powerful way to extend the functionality of a generator, enabling dynamic behavior, custom workflows or 
generating code for different languages. By developing your own plugin, you can tailor the generator to suit specific 
needs, such as processing custom inputs, providing options, or integrating with external systems.

This guide will walk you through the steps of creating your own plugin, covering everything from adding dependencies to 
handling specific signals and sending back appropriate responses. Whether you are building a plugin to modify the behavior of an existing generator or to introduce new options, this tutorial will help you get started with the Plugin API.

## Step 1: Add Dependencies

Before starting development, you need to add the necessary dependencies for the plugin to interact with the host system and exchange data. Ensure that your project includes the required libraries for communication via the Plugin API.

In your `build.gradle.kts` file (for Gradle projects), include the following dependencies:
```Kotlin
dependencies {
    implementation("org.timemates.rrpc:generator-core:$latestVersion")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.0")
}
```
Ensure you use the correct version numbers according to your setup.

## Step 2: Define the Adapter

The adapter is used to map the available options and process the input received from the generator. Its purpose
is to generate code, based on the input received:
```Kotlin
public interface SchemaAdapter {
    /**
     * List of available options for schema adapter with description. For CLI and Gradle plugin.
     */
    public val options: List<GenerationOption>

    /**
     * This method is used for generating the code usually, but can be used for other
     * purposes, for an example, – logging.
     *
     * @return RMResolver that might be the same as [resolver] or new in cases
     * when you want to modify incoming data to the following adapters.
     */
    public fun process(
        options: GenerationOptions,
        resolver: RSResolver,
    ): RSResolver
}
```
For the reference, you might want to see the implementation of [Kotlin Code Generator](https://github.com/timemates/rrpc-kotlin/blob/0.7.0/generator/kotlin/src/commonMain/kotlin/org/timemates/rrpc/generator/kotlin/adapter/KotlinSchemaAdapter.kt).

<warning>
This section is yet to be finalized.
</warning>

Once you implemented the `SchemaAdapter` for your needs, we can continue to the setup of communication between Generator
and the Plugin:

## Step 3: Setup Plugin Communication
The next step is setting up the communication between your plugin and the generator. 
The plugin will send and receive signals, which are the core units of communication.

To begin, create an instance of the PluginCommunication class to handle the input and output streams:
```Kotlin
val client = PluginCommunication(
    System.`in`.source().buffer(),    // Input stream from the generator
    System.out.sink().buffer()        // Output stream to the generator
)
```

> [```rrcli```](CodeGen-CLI.md) as well as a Gradle Plugin, communicate with plugins through stdin/stdout. In the
> example we will use java.lang.System to obtain it, but it's not locked up – we use okio for ability to target non-JVM
> targets.

## Step 4: Handling Incoming Signals
When your plugin receives a signal, it processes the signal based on its type. A plugin will always receive a G
eneratorSignal.FetchOptionsList at the very beginning, which requests the available options. 
In response, you can send a list of options back to the generator.

The generator might also send a GeneratorSignal.SendInput, but it is up to the generator to decide whether this signal 
is needed, for example, when requesting input for certain options, such as --help.

Here’s an example of how to handle these signals:
```Kotlin
client.receive { signal ->
    when (signal) {
        GeneratorSignal.FetchOptionsList -> listOf(
            PluginSignal.SendOptions(
                schemaAdapter.options.map { it.toOptionDescriptor() }
            ),
            PluginSignal.RequestInput,
        )
        is GeneratorSignal.SendInput -> {
            schemaAdapter.process(
                options = GenerationOptions(signal.args), 
                resolver = RSResolver(signal.files),
            )
            emptyList()  // No further response needed
        }
    }
}
```
In this example:
- GeneratorSignal.FetchOptionsList is always received and is responded to by sending a list of available options and potentially a RequestInput signal.
- GeneratorSignal.SendInput may or may not be received, depending on the generator’s requirements.

## Step 5: Testing the Plugin

Before deploying your plugin, it’s essential to test it to ensure that it reacts correctly to signals and sends the appropriate responses. You can use unit tests or a local testing environment to simulate the interactions between your plugin and the generator.

<warning>
This section is yet to be finalized.
</warning>

## Conclusion

By following these steps, you can develop a plugin that communicates with the generator using the Plugin API. The key steps include adding dependencies, setting up communication, handling signals (particularly FetchOptionsList), and sending appropriate responses. This allows your plugin to seamlessly integrate with the generator’s functionality.

For more details and further examples, refer to the Plugin API documentation.
