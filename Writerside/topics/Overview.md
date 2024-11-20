# Overview

## What is rRPC?

rRPC abbreviation simply stands for RSocket + RPC. This framework is designed to expose APIs as RPC (Remote Procedure Call)
Services, enabling the creation of gRPC-like services directly from `.proto` files through code generation. rRPC provides
essential core components for both server and client-side development, making it easier to build scalable and efficient
RPC services.

## Why Not gRPC or Other Solutions?

While gRPC is a widely adopted solution for RPC services, it comes with certain limitations, particularly for Kotlin
Multiplatform development. Currently, gRPC does not have near-future support for Kotlin Multiplatform, and though
alternatives like the 'wire' library exist, they require significant manual adaptation between platforms. This approach,
although easier than managing entirely separate codebases, still presents challenges.

Additionally, gRPC has several limitations, especially in web environments where it lacks support for bidirectional
streaming. This limitation complicates the development process when full-duplex communication is needed.

rRPC, on the other hand, is designed with simplicity in mind. Unlike gRPC, it does not require inventing new
communication mechanisms or dealing with complex request schemas. It reuses existing communication model (RSocket), what
makes it easier in adopting and supporting.Developers can even write clients using plain RSocket and Protobuf
serialization libraries, making the framework more accessible and easier to implement.

## Why Choose rRPC?

The primary goal of rRPC is to support a wide range of platforms and languages. Currently, the framework supports
Kotlin (at the moment of prototyping, for convenience), with plans to extend support to Java (via a bridge), JavaScript,
and Python in the near future. The rRPC team is actively seeking feedback before the 1.0.0 release to refine the
framework's mental model and deliver a stable, user-friendly solution.

## Why Protobuf?

rRPC is tightly integrated with Protobuf for serialization. We believe it's the most efficient in terms of compactness
and versioning for now. In addition, it's one of the most popular formats. While supporting other serialization
mechanisms could be
beneficial, the team believes that sticking with Protobuf ensures better compatibility and understanding across
different platforms. Introducing multiple serialization mechanisms could lead to compatibility issues and increased
complexity in support, which is why the focus remains on Protobuf.

