# Implementation Details
If you're interested, how rRPC works internally, but don't want to browse code, this section is for you.

## Metadata
As you already know, code is generated and then serialized using ProtoBuf. In RSocket requests, there are two parts that
are sent/received: metadata and data. If we already know what encoded in the data (it's actually type specified in schema),
in metadata, we encode the following:
```
message ClientMetadata {
    int32 schemaVersion = 1;
    string serviceName = 2;
    string procedureName = 3;
    map<string, bytes> extra = 4;
}

message ServerMetadata {
    int32 schemaVersion = 1;
    map<string, bytes> extra = 2;
}
```

Let's dive into `ClientMetadata`:
- `schemaVersion`: version of the schema, for compatibility purposes only (for now unused, version = 1).
- `serviceName`: fully qualified name from `.proto` file – package name + service name
- `procedureName`: name of the rpc in the scheme.
- `extra`: custom metadata to be passed to server/client by interceptors and so on.

For `ServerMetadata` it's the same.

## Requests
There are five types of requests in RSocket: Request-Response, Request-Stream, Request-Channel, Fire-And-Forget 
and Metadata-Push.

We support all types of those requests, but Fire-And-Forget and Metadata-Push are experimental:

| proto                                                 | Request Type     |
|-------------------------------------------------------|------------------|
| `rpc (T) returns (R)`                                 | Request-Response |
| `rpc (T) returns (stream R)`                          | Request-Stream   |
| `rpc(stream T) returns (stream R)`                    | Request-Channel  |
| `rpc(timemate.rrpc.Ack) returns (R)`                  | Fire-And-Forget  |   
| `rpc (timemate.rrpc.Ack) returns (timemate.rrpc.Ack)` | Metadata-Push    |

Client-only streaming is not supported by transport (RSocket), so we don't support it either. We support other
request types by annotation type called `Ack`, here's [an issue](https://github.com/timemates/rrpc-kotlin/issues/9) to discuss.


___________________________________


That's all what you probably need to know. Implementation is very simple and requires no many efforts.





