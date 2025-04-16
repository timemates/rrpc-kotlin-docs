# Gradle Plugin

The rRPC Gradle Plugin helps automate code generation for rRPC services in Gradle-based projects using `.proto` files.

<note>
If your project is not Gradle-based, you can use the <a href="Tooling-CLI.md">rrgcli</a> for generating code.
</note>

## Applying the Plugin

Add the plugin to your `build.gradle.kts`:

```kotlin
plugins {
    id("app.timemate.rrpc") version "$latestVersion"
}
```

<note>
Make sure that you have Gradle Plugin Portal specified in your repositories.
</note>

## Configuring the Plugin

The plugin is configured using the rrpc extension, allowing customization for .proto file locations, output
configurations, and generation behavior.

### Basic Example

```Kotlin
rrpc {
    /**
     * Defines the input sources for the RRpc code generation.
     *
     * The input sources are categorized into two types:
     * - **Source**: The primary proto files to generate the RRpc code from.
     * - **Context**: Artifacts used for resolving references. If a reference is encountered during code generation,
     *   the context files will be processed to include the necessary references.
     */
    inputs {
        /**
         * Directory containing the source proto files. These files will be directly processed for code generation.
         * The specified directory is the primary source for RRpc code generation.
         */
        source {
            directory(project.file("src/main/proto"))
        }

        /**
         * Artifact dependencies used for resolving references during code generation.
         * Only files that are actually referenced will be processed and included in the generated output.
         */
        context {
            // Add it in case you use rrpc builtin types like Ack or 
            // custom options.
            artifact(libs.timemates.rrpc.common)
        }
    }

    /**
     * Defines the base output folder for the generated RRpc code.
     * Each plugin used in the generation process will create a subfolder inside this output folder.
     * The name of each subfolder corresponds to the plugin name.
     *
     * Example: If the `kotlin-generator` plugin is used, the output for it will be in
     * the folder named after that plugin inside this output folder.
     */
    outputFolder = layout.buildDirectory.file("generated/rrpc")
        .get().asFile

    // whether is to allow package cycles within proto sources
    permitPackageCycles = true

    /**
     * Specifies the plugins that will be used during the RRpc code generation process.
     * Each plugin is identified by its Maven coordinate, and it is automatically resolved
     * (including downloading the necessary binary) when needed.
     *
     * In this case, the `kotlin-generator` plugin is added with its version set to `SNAPSHOT`.
     * This plugin will handle the generation of Kotlin code for RRpc.
     */
    plugins {
        add("app.timemate.rrpc:kotlin-generator:SNAPSHOT") {
            // default values for generator that can be overwritten
            option("server_generation", true)
            option("client_generation", true)
            option("type_generation", true)
            option("metadata_generation", true)
            // 
            option("metadata_scope_name", "FooBar")
            // whether is to adapt naming of proto members with '_' to
            // kotlin's camel case
            option("adapt_names", true)
            // whether is to apply 'data' modifier to messages
            // might be useful for debugging or testing
            // for production, especially for js target,
            // it's recommended to disable this parameter
            option("message_data_modifier", true)
        }
    }
}
```

#### Plugins
Every option, except the basic one that is available within Gradle Plugin DSL (like `permitPackageCycles`)
are scoped per plugin. In the case of Gradle Plugin, you need to use special DSL as in the example.

<warning>
If you have complex setup, where some plugin may override the input, ensure the correct order of
plugins.
</warning>

It's worth mentioning that plugin can be published as binary to the maven, in that case to automatically assign
classifier
while resolving, do the following:

```kotlin
plugins {
    add("group:artifact:version", type = PluginDependencyType.BINARY)
}
```

##### File system dependencies
In case if plugin is not distributed via maven (or simply made not with JVM specifics),
you may add it locally with the same ability to auto-assign classifier:
```Kotlin
plugins {
    add(files("libs/plugin-name"), type = PluginDependencyType.BINARY)
}
```
By doing like that, it will auto-assign the following depending on the OS:
- For Windows: plugin-name-windows-x86_64.exe
- For Linux: plugin-name-linux-x86_64 (without extension)
- For MacOS: plugin-name-macos-aarch64 (without extension)

If you don't rely on auto-assigning principle, you may just not add an explicit type of plugin.

##### Local module dependencies
We also support ability to be dependent on your module as a source of the plugin. To add it make the following:
```kotlin
plugins {
    add(project(":plugin-module"))
    // or when using project safe accessors
    add(projects.pluginModule)
}
```
Take into account that plugins are run in a separate process and don't have runtime classpath
due to the nature of Gradle itself.
Therefore, you need to ensure that it's presented. For example, you may use shadow jar:

```kotlin
application {
    mainClass.set("...")
}
// --------------------------------------
// ShadowJar Configuration
// --------------------------------------
tasks.shadowJar {
    // to overwrite default output
    archiveClassifier.set("")
}

artifacts {
    add("archives", tasks.shadowJar)
}

configurations {
    named("runtimeElements") {
        outgoing.artifacts.clear()
        outgoing.artifact(tasks.shadowJar)
    }
    named("apiElements") {
        outgoing.artifacts.clear()
        outgoing.artifact(tasks.shadowJar)
    }
}
```
It's the way, for example, how we test our `rrpc-kotlin-gen` plugin in our rrpc-kotlin repository.

We explore better ways to handle it and if you're expert in Gradle and have better solution, you're welcome as a contributor!

#### Dependencies
For now, you can't exclude dependencies from generation, instead, you should rely
on generation on the last module in the chain.
Meaning, if some other module you're dependent on,
and it generates a certain type you reuse on another side, you will run into
conflicts.

We're investigating how we can implement it at the moment in the best way possible from API side;
you may refer to the related [issue](https://github.com/timemates/rrpc-generator/issues/2).

#### Source sets
As from the design point, we don't know which plugins you will apply, you may need to specify it
correctly for your build to work, for example, how it should be done with `rrpc-kotlin-gen` plugin:
```kotlin
sourceSets {
    getByName("main") {
        kotlin.srcDirs(
            "src/main/kotlin",
            "build/generated/rrpc/rrpc-kotlin-gen",
        )
    }
}
```

#### Running plugin
There are two tasks available for you to use:
- `./gradlew :generateRRpcCode`: to generate the code
- `./gradlew :rrpcGeneratorHelp`: to get help with all possible parameters for different plugins.
