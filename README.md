# GRPC, Meeting #1

## Notes from the meeting

* What gRPC is and why/how it is used

  * It's a solution for the problem of communicating a stable, vesatile, lean API contract between service providers/consumer, esp. in an architecture composed of many networked (micro)services

  * gRPC vs REST (vs others, e.g. GraphQL)
    * gRPC works very well with protocol buffers, but does not actually require them.
      Neither protobuf nor binary encoding across the wire is, technically speaking, a differential between gRPC and REST.
      You *can* use gRPC with JSON, or REST with protobuf (though neither is very common).
      gRPC has a reputation for being more performant and type-safe than REST, but whatever help it gets from protobuf on those advantages is not an inherent quality of gRPC.
      * Interesting, though dated comparison of gRPC vs REST / protobuf vs JSON:  [Sending Data to the Other Side of the World: JSON vs Protocol Buffers and REST vs gRPC](https://www.kabisa.nl/tech/sending-data-to-the-other-side-of-the-world-json-protocol-buffers-rest-grpc/)
    * gRPC requires a shared RPC contract between a server and each of its clients; this constitutes the 1st half of a shared interface contract between client & server that is completed by protobuf types.
      REST would typically benefit from a shared contract, but can't enforce it with JSON, and tends to use API docs instead.
    * gRPC is implemented on top of HTTP/2. This adds two critical advantages to gRPC:
      * the inherent performance boost from HTTP/2
      * the ability to implement *streaming* RPC interfaces
![001](img/grpc/001.png)

    * gRPC tends to be used internally; the overhead of sharing the service contract with the client is not geared toward exposing a public interface.
      REST (or alternatives, e.g. GraphQL) will tend to be the public interface, with a translation layer to gRPC internally.
    * REST APIs can be versioned and maintained to provide backward compatibility.
      But gRPC was designed from the start with backward compatibility as a core strength.

  * Protobuf vs JSON
    * [Protocol buffers](https://developers.google.com/protocol-buffers) are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler.
    * Protobuf is theoretically about data serialization, not necessarily about supporting gRPC. In practice, that's its typical use. But like JSON or xml, protobuf could be used to represent data in a way not limited to interface description.
    * Protobuf serializes typed data; the types constitute the 2nd half of a shared interface contract between client & server.
    * Protobuf is binary over the wire; JSON isn't. That means protobuf is leaner than json, but probably comparable or slightly less lean than compressed JSON.
    * Protobuf works seamlessly with gRPC via compilers that generate service stubs for easy implementation. It's possible to use protobuf with REST, but less common.
    * Protobuf uses namespace packaging and supports data-definition sharing via imports.

  * Fundamentals of how gRPC works
    * Message schemas are defined in a `.proto` file.

          // The greeter service definition.
          service Greeter {
            // Sends a greeting
            rpc SayHello (HelloRequest) returns (HelloReply) {}
          }

          // The request message containing the user's name.
          message HelloRequest {
            string name = 1;
          }

          // The response message containing the greetings
          message HelloReply {
            string message = 1;
          }

    * The protoc (or equivalent) compiler (perhaps + plugins) is then used to generate a language specific implementation of the message schema and Server + Client "stubs".
![0012](img/grpc/0012.png)

  * Proto2 vs proto3
    * *(See [Protocol Buffers v3](https://cloud.google.com/apis/design/proto3))*
    * proto3 does not support required vs optional fields. All fields in proto3 are optional.
    * proto3 does not support specifying default values for optional fields.
      * The default value is fixed for type (it is usually 0, false, empty string, etc.)
      * It is impossible to distinguish absent fields from default values... without a work-around
        * [Truly optional scalar types in protobuf3 (with Go examples)](https://iximiuz.com/en/posts/truly-optional-scalar-types-in-protobuf3/)
        * [Oneof Fields](https://developers.google.com/protocol-buffers/docs/reference/go-generated)

  * Resources, links
    * online docs
      * [Introduction to gRPC](https://grpc.io/docs/what-is-grpc/introduction/)
      * [Core concepts, architecture and lifecycle](https://grpc.io/docs/what-is-grpc/core-concepts/)
    * book
      * [gRPC Up & Running](https://www.amazon.com/gRPC-Running-Building-Applications-Kubernetes/dp/1492058335), ch. 1 (Note: This is available on <https://learning.oreilly.com/>. Ping me for access if you want/need it.)
    * Pluralsight courses (Onshore: Check with Todd MacQueen if you need access to ITT's Pluralsight licenses. Offshore: I don't know offhand what the Pluralsight licensing situation is; check with you manager and remind him/her of the high ROI for developer training if you need access.)
      * [Moving Beyond JSON and XML with Protocol Buffers](https://app.pluralsight.com/library/courses/protocol-buffers-beyond-json-xml/table-of-contents), Pluralsight course by Mike Van Sickle
      * [Enhancing Application Communication with gRPC](https://app.pluralsight.com/library/courses/grpc-enhancing-application-communication/table-of-contents), Pluralsight course by Mike Van Sickle

* Ecosystem/Tools/Implementation/Standards

  * gRPC communication with protobuf encoding:
    * gRPC implementation layers include gRPC core that abstracts the networking and encoding implementation + server/client stubs compiled into target language for consumption
      * [gRPC-Go](https://github.com/grpc/grpc-go)
![0015](img/grpc/0015.png)

    * HTTP/2 connection
      * gRPC uses HTTP/2 as its transport protocol: One of the reasons why gRPC is high-performance.
        All communication between a client and server is performed over a single TCP connection that can carry any number of bidirectional flows ("streams") of bytes.
![002](img/grpc/002.png)
![003](img/grpc/003.png)

    * headers/trailers: [gRPC over HTTP2](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md#grpc-over-http2)
![004](img/grpc/004.png)
![005](img/grpc/005.png)

    * [metadata](https://grpc.io/docs/what-is-grpc/core-concepts/#metadata)
      * *exercise: spot the metadata in the example request headers [here](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md#grpc-over-http2)*
      * [grpc-go docs on metadata](https://github.com/grpc/grpc-go/blob/master/Documentation/grpc-metadata.md)
    * [official gRPC response codes](https://grpc.github.io/grpc/core/md_doc_statuscodes.html)
      * See also: <https://pkg.go.dev/google.golang.org/grpc/codes>

  * Creating protobuf message and service definitions
    * follow best practices
      * style guids: Strongly advise reading through both of these and making some notes
        * [Google](https://developers.google.com/protocol-buffers/docs/style)
        * [Uber](https://github.com/uber/prototool/blob/dev/style/README.md)

  * Updating, versioning and maintaining backwards compatibility
    * Good advice from the [Uber style guide](https://github.com/uber/prototool/blob/dev/style/README.md#reserved-keyword): "Your API as a whole should not need semantic versioning - one of the core promises of Protobuf is forwards and backwards compatibility, and this should extend to your code as well. This is especially true if your Protobuf definitions are used across multiple repositories, and even if you have a single monorepo for your entire organization, any external customers who use your Protobuf definitions should not be broken either."
    * "deprecated" (and "reserved") keywords
    * version-rolling your proto definitions

  * Compiling the protobuf into client/server stubs
    * compilers
      * [Protoc](https://grpc.io/docs/protoc-installation/) *(not used directly at kount)*
      * [Prototool](https://github.com/uber/prototool) *(no longer used by Kount, and, indeed, not even recommended now by Uber, who created it)*
      * [Buf](https://github.com/bufbuild/buf)
        * See also: The [Buf docs](https://docs.buf.build/introduction)
    * compiler plugins
      * [Go support for protobuf](https://pkg.go.dev/google.golang.org/protobuf#section-readme) requires a compiler plugin ([protoc-gen-go](https://pkg.go.dev/google.golang.org/protobuf/cmd/protoc-gen-go)) and a runtime library. See <https://developers.google.com/protocol-buffers/docs/reference/go-generated#invocation>.
      * [Buf requires](https://docs.buf.build/tour/generate-go-code#install-plugins) `protoc-gen-go` and `protoc-gen-go-grpc` plugins to generate Go code.
      * grpc-gateway requires the `protoc-gen-grpc-gateway` plugin
      * For example, a generic example `buf.gen.yaml` needs multiple plugins to compile Go:

            version: v1
            plugins:
              - name: go
                out: gen/go
                opt:
                  - paths=source_relative
              - name: go-grpc
                out: gen/go
                opt:
                  - paths=source_relative
              - name: grpc-gateway
                out: gen/go
                opt:
                  - paths=source_relative
                  - generate_unbound_methods=true

  * Unary vs streaming gRPC // Todo: meeting #2

  * Middleware/Interceptors // Todo: meeting #2

  * gRPC gateway // Todo: meeting #2

  * Tools // Todo: meeting #2

* Resources, links
  * [Protocol Buffer Basics: Go](https://developers.google.com/protocol-buffers/docs/gotutorial)
  * [Proto3 Language Guide](https://developers.google.com/protocol-buffers/docs/proto3)
  * Protobuf [well-known types](https://developers.google.com/protocol-buffers/docs/reference/google.protobuf)
  * [Go Generated Code](https://developers.google.com/protocol-buffers/docs/reference/go-generated)

* Kount implementation and standards/patterns // Todo: meeting #2
  * details
  * {example(s), demo, exercise(s)}
  * Resources, links

* Common techniques & tasks, "gotchas", tips & tricks // Todo: approx. meeting #3-4

* {Case Studie(s)} // Optional: maybe meeting #5

* {"Class" exercise(s)} // Todo: approx. meeting #5-6
