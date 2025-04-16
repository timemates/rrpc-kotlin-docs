Got it. Here's the revised version of the **`rrgcli` documentation page**, tailored to your request and aligned with your documentation tone and format:

---

# CLI

The `rrgcli` tool allows you to generate code from `.proto` files for use with the rRPC framework. It supports creating client stubs, server stubs, and data models through an extensible plugin system.

<note>
If you're using Gradle, consider using the <a href="Tooling-Gradle.md"/> for a more integrated experience.
</note>

## Installation

Download the latest release of `rrgcli` from the [rRPC GitHub Releases](https://github.com/timemates/rrpc-kotlin/releases).  
After downloading, make it executable and move it to your PATH for convenience:

```bash
chmod +x rrgcli
mv rrgcli /usr/local/bin
```

## Usage

The CLI is plugin-driven. Each plugin exposes its own scoped options, which can be passed via the command line using the format:

```
--<plugin-name>:<option-name>=<value>
```

You can get the plugin name by inspecting its documentation or running the plugin directly with `--help`. Basic options
like `--protos_input` do not follow this pattern.

<warning>
The example below demonstrates usage with `rrpc-kotlin-gen`, a Kotlin generator plugin.  
This plugin is **not bundled by default** with `rrgcli`. You can obtain it from the <a href="https://github.com/timemates/rrpc-kotlin/releases">rRPC GitHub Releases</a>. It is provided here **for reference only**.
</warning>

Example `--help` output when using `rrgcli` with the Kotlin generator plugin:

```
Usage: rrgcli [<options>]

Options:
  --protos_input=<text>   Folder with `.proto` files to be used for generation
                          (repeatable).
  --permit_package_cycles=true|false
                          Indicates whether package cycles should be ignored
                          while parsing `.proto` files.
  --rrpc-kotlin-gen:client_generation=true|false
                          Generate Kotlin client stubs.
  --rrpc-kotlin-gen:server_generation=true|false
                          Generate Kotlin server stubs.
  --rrpc-kotlin-gen:type_generation=true|false
                          Generate Kotlin data types.
  ...
  -h, --help              Show this message and exit.
```

See also: [Kotlin Generator Options](Kotlin-Generation-Plugin.md)

## Including a Plugin

To include a plugin in the generation process, use the `--plugin` option:

```bash
rrgcli --plugin=path/to/plugin
```

If the plugin is a `.jar` file or a command that launches a plugin-compatible process, you can also pass it like this:

```bash
rrgcli --plugin="java -jar path/to/plugin.jar"
```
<tip>
The plugin architecture allows combining multiple generators or preprocessors (e.g., to filter or transform `.proto` input).
</tip>

## Plugin Options

Plugins define their own configuration options. These options are scoped by the plugin's name using the format:

```
--<plugin-name>:<option>=<value>
```

You can discover available plugin-specific options by checking the plugin's documentation or running it with `--help`.

## Examples

### Generate Client Stubs and Types (with Kotlin Generator Plugin)

```bash
rrgcli \
  --protos_input=protos \
  --rrpc-kotlin-gen:client_generation=true \
  --rrpc-kotlin-gen:type_generation=true
```

### Multiple `.proto` Input Directories

```bash
rrgcli \
  --protos_input=proto/common \
  --protos_input=proto/service \
  --rrpc-kotlin-gen:server_generation=true
```
<seealso>
    <a href="Custom-Plugins.md" type="idea" summary="Learn how develop your own plugins.">Plugin development</a>
    <a href="Kotlin-Generation-Plugin.md" type="idea">How to use Kotlin Generation Plugin</a>
</seealso>