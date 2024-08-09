# Compatibility with ProtoBuf

Our implementation of ProtoBuf serialization supports a subset of ProtoBuf features. Below is a detailed overview of the supported and unsupported features, organized by category.

## Unsupported ProtoBuf Features

1. **Types**:
    - **`uintXX` and `sfixedXX`**: Not supported due to limitations in `kotlinx.serialization.protobuf`.

2. **Options**:
    - **File and Message Options**: Not supported. For use cases requiring these features, please [open an issue](#).
    - **Option Retention**: Not supported. Options cannot be preserved across serialization/deserialization.
    - **Option Definition Inside Message Construct**: Not supported. Options must be defined externally.
    - **`bytes` Type for Options**: Not supported. This includes `map<bytes, bytes>`. If needed, please [open an issue](#).

3. **Other Features**:
    - **`google.protobuf.Any`**: Not supported. Arbitrary message types cannot be embedded.
    - **Reserved Fields**: Not supported. Reserved fields are ignored.
    - **Extensions**: Not supported. Custom fields must be defined within standard message structures.
    - **Proto2**: Not supported. Only Proto3 syntax is supported.

## Supported ProtoBuf Features

1. **Scalar Types**:
    - All scalar types are supported except `uintXX` and `sfixedXX`. This includes `string` and other standard types.

2. **Complex Types**:
    - **Maps and Repeated Fields**: Fully supported, allowing for flexible data structures.
    - **Nested Messages**: Fully supported, including messages within messages.
    - **Enums**: Supported and mapped to Kotlin enums for straightforward handling.

3. **Alternative Fields**:
    - **Oneofs**: Supported through Kotlin sealed classes, enabling safe handling of alternative field values.

4. **Wrapper Types**:
    - Supported through Kotlin primitives, but nullable. This approach may be reconsidered in future updates (until 1.0).

5. **Custom Options**:
    - Supported, provided the type is not `bytes`. Other types for custom options are handled.
6. **Default Values**:
    - Supported only for options. Default values for other fields are not managed as it's not supported by protobuf 3.

7. **Generated Protos**:
    - Protos generated from `.proto` files are supported similarly to user-defined protos.

For features not supported or additional requirements, please [open an issue](https://github.com/timemates/rsp/issues/new/choose) to discuss potential support or contributions. This will help us understand and potentially address your use cases.

You may also want to learn more about options and how they're handled [here](ProtoBuf-Options.md).