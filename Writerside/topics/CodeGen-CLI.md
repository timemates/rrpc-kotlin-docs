# CLI
The `rrgcli` tool allows you to generate code from `.proto` files for use with the rRPC framework. It supports creating
Kotlin-based (and other, in future) client stubs, server stubs, data types, and metadata. 

<note>
If you're using Gradle, it's a much more convenient to use <a href="CodeGen-Gradle.md"/> for generating the code.
</note>

## Installation

Download the latest release of `rrgcli` from the [rRPC GitHub Releases](https://github.com/timemates/rrpc-kotlin/releases).
Once downloaded, ensure the binary is executable and optionally add it to your system's PATH:

```Bash
chmod +x rrgcli
mv rrgcli /usr/local/bin
```
## Usage
The tool processes .proto files and outputs generated code based on the options you provide.
Use the --help option to see available parameters:
```
Usage: rrgcli [<options>]

Options:
  --protos_input=<text>   Folder with `.proto` files to be used for generation
                          (repeatable).
  --permit_package_cycles=true|false
                          Indicates whether package cycles should be ignored
                          while parsing `.proto` files.
  --kotlin_output=<text>  Specifies the output path for generated Kotlin files.
  --kotlin_client_generation=true|false
                          Indicates whether client stubs should be generated
                          for Kotlin. Default is `false`.
  --kotlin_server_generation=true|false
                          Indicates whether server stubs should be generated
                          for Kotlin. Default is `false`.
  --kotlin_type_generation=true|false
                          Indicates whether data types should be generated for
                          Kotlin. Default is `false`.
  --kotlin_metadata_generation=true|false
                          Specifies whether metadata should be generated.
  --kotlin_metadata_scope_name=<text>
                          Specifies the scope name for metadata generation. If
                          not specified, a global instance of Metadata Container
                          is used.
  -h, --help              Show this message and exit.
```
> **Additionally about metadata generation**
> 
> Enabling the `--kotlin_metadata_generation` flag generates debugging information about the whole schema, meaning that,
> factually, it provides the same information as any generator have on generation time, but on app's runtime. 
> This can be helpful for servers that need details about options attached to specific levels (e.g., file-level options for a service declaration). Metadata may also provide insight into the
> organization and layout of services for tooling.
> 
>
> The `--kotlin_metadata_scope_name` flag is used to make metadata scoped to a specific instance rather than placing it 
> in the global object. This allows you to:
> - Filter out unnecessary schema information from your SchemaService.
> - Avoid polluting the global scope with unrelated or redundant metadata.
> - Tailor metadata for specific use cases or service groups.
> 
> By scoping metadata, you can maintain cleaner separation of concerns, particularly in larger projects with diverse requirements.


## Examples
### Generate Client Stubs and Types
This example generates Kotlin client stubs and data types from .proto files located in the protos directory:
```Bash
rrgcli \
  --protos_input=protos \
  --kotlin_output=build/generated \
  --kotlin_client_generation=true \
  --kotlin_type_generation=true
```

### Handle Package Cycles and Metadata
This example resolves package cycles in .proto files and generates metadata with a custom scope name:
```Bash
rrgcli \
  --protos_input=protos \
  --kotlin_output=build/generated \
  --permit_package_cycles=true \
  --kotlin_metadata_generation=true \
  --kotlin_metadata_scope_name=my_custom_scope
```

### Multiple `.proto` Input Directories
To include .proto files from multiple locations:
```Bash
rrgcli \
  --protos_input=protos1 \
  --protos_input=protos2 \
  --kotlin_output=build/generated \
  --kotlin_server_generation=true
```
_______
