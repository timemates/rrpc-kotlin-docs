# Kotlin Generation Plugin

The Kotlin Generation Plugin for `rrgcli` and the Gradle CodeGen Plugin allows you to generate idiomatic Kotlin code from `.proto` files for use with the rRPC framework. This includes generating client stubs, server stubs, shared data types, and optionally metadata used for tooling or runtime schema inspection.

This plugin is **not bundled** with `rrgcli` by default and must be provided manually using the `--plugin` option or as a dependency in the Gradle configuration.

## Plugin Capabilities

The Kotlin generator supports:

- Generating **client stubs** to call rRPC services from Kotlin applications.
- Generating **server stubs** to implement rRPC services.
- Generating **type-safe data models** from `.proto` definitions.
- Generating **schema metadata** to inspect service layouts or options at runtime (optional).
- Optional **name adaptation** to generate idiomatic Kotlin names while preserving mapping to the original `.proto`.

## Usage with `rrgcli`

To invoke the Kotlin plugin via CLI, specify the plugin using the `--plugin` flag, and pass any required options with the `rrpc-kotlin-gen:` prefix:

```bash
rrgcli \
  --source_input=/path/to/protos \
  --gen_output=./output \
  --plugin="java -jar /path/to/kotlin-generator.jar" \
  --rrpc-kotlin-gen:client_generation=true \
  --rrpc-kotlin-gen:server_generation=false \
  --rrpc-kotlin-gen:type_generation=true
```

<warning>
This plugin must be explicitly specified using --plugin, as it is not bundled with rrgcli by default.  
You can download the Kotlin Generator from the <a href="https://github.com/timemates/rrpc-kotlin/releases">rRPC Kotlin GitHub Releases</a>.
</warning>

<note>
To learn more about installing and using the CLI tool, see <a href="Tooling-CLI.md">rRPC CLI Tool</a>.
</note>

### Available CLI Options

The following options are scoped under `rrpc-kotlin-gen:` when used with `rrgcli`:

- `client_generation=true|false`  
  Whether to generate Kotlin client stubs. Defaults to `false`.

- `server_generation=true|false`  
  Whether to generate Kotlin server stubs. Defaults to `false`.

- `type_generation=true|false`  
  Whether to generate Kotlin data models. Defaults to `false`.

- `metadata_generation=true|false`  
  Whether to include schema metadata in the generated output. Useful for runtime inspection and tools.

- `metadata_scope_name=<text>`  
  Assigns a scope name to the generated metadata. If omitted, metadata is placed in a global container.

- `adapt_names=true|false`  
  If set to `true`, field and type names will be adapted to Kotlin naming conventions. Defaults to `true`.

- `message_data_modifier=true|false`
  If set to `true`, messages will be `data` classes. Especially useful for logging and testing purposes. Defaults to `true`.

<note>
See <a href="Kotlin-Metadata-Generation.md"/> to learn how and when to enable metadata.
</note>

## Usage with Gradle

The Kotlin generator can also be used seamlessly with the Gradle CodeGen plugin, providing a declarative and composable configuration via DSL.

```kotlin
rrpc {
    inputs {
        source {
            directory(project.file("src/main/proto"))
        }

        context {
            artifact(libs.timemates.rrpc.common.get())
        }
    }

    permitPackageCycles = false
    outputFolder = layout.buildDirectory.file("generated/rrpc")

    plugins {
        add(
            notation = "app.timemate.rrpc:kotlin-generator:SNAPSHOT",
            type = PluginDependencyType.JAR,
        ) {
            option("server_generation", false)
            option("client_generation", true)
            option("type_generation", true)
            option("adapt_names", true)
        }
    }
}
```

In Gradle:

- Plugin options are passed via the `option(name, value)` DSL method.
- You don't need to prefix options with `rrpc-kotlin-gen:` — scoping is handled automatically.
- You can declare plugin dependencies using Maven coordinates or file paths.

<note>
See <a href="Tooling-Gradle.md">Gradle Plugin Documentation</a> for full configuration examples.
</note>

## Name Adaptation

By default, the plugin will adapt `.proto`-defined names to follow Kotlin naming conventions. For example:

- `user_id` → `userId`

If you want to preserve original names exactly as written in `.proto`, disable adaptation with `adapt_names` to `false`.

## Metadata Generation

When `metadata_generation` is enabled, the plugin will generate a snapshot of the entire rRPC schema. This snapshot can be accessed at runtime or used for tooling purposes.

If you have multiple modules that generate metadata, you can isolate different metadata modules using the `metadata_scope_name` option to avoid polluting a global registry.

<note>
Metadata generation is <b>optional</b> and most likely should only be used for server tooling, admin dashboards, or advanced validation use cases.
Read more in <a href="Kotlin-Metadata-Generation.md"/>.
</note>

## Summary

The Kotlin plugin brings first-class support for generating Kotlin-first APIs, models, and infrastructure. Whether you prefer working from the command line or using a build system like Gradle, the plugin provides flexible integration points and a clear separation of concerns through its options and plugin-scoped configuration.