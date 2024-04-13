# gRPC and Go!
A quick tour of how to easily utilize gRPC in your Go projects

## Agenda
- Intro to gRPC and Protobufs
- gRPC & Protobufs vs REST & JSON
- When to use gRPC and REST
- Considerations When Using gRPC
- A tour of Buf
- Demo
- Buf at Spectro Cloud

Feel free to chime in with comments or questions at any time during the presentation

---

## Intro to gRPC and Protobufs
- Protobufs (protocol buffers) is a language agnostic data serialization format
    - Desinged to efficiently serlialize structured data into compact binary format


- gRPC is an open source RPC (remote procedure call) framework
    - Uses protobuf as its default serialization format

Think of gRPC as the highway used to transport your data, with protobufs acting as the super efficient cars to pack your data into

---

## gRPC & Protobufs vs REST & JSON
- REST will use the standard HTTP methods we all know
- Your JSON will get serialized and transfered with HTTP

```json
{
    "userName": "Martin",
    "favouriteNumber": 1337,
    "interests": ["daydreaming", "hacking"]
}
```
- When serialized, everthing from the key names to the value needs to be encoded leading to "heavy" payloads
- Serialized into 95 bytes

---

## gRPC & Protobufs vs REST & JSON
- Your data will get serialized and transfered with HTTP/2
- A `.proto` file needs to be created defining the message schema and the service endpoints available
- Next you'll need to generate the gRPC client and server interfaces and types using `protoc`

### proto definition
```proto
message PersonRequest {
    string user_name          = 1;
    int64 favorite_number     = 2;
    repeated string interests = 3;
}

message PersonResponse {}

service PersonService {
    rpc AddPerson(PersonRequest) returns (PersonResponse) {}
}
```

### go client code
```go
// Create a new PersonRequest.
req := &pb.PersonRequest{
    UserName:       "Martin",
    FavoriteNumber: 1337,
    Interests:      []string{"daydreaming", "hacking"},
}

// Call the AddPerson method.
res, err = client.AddPerson(req)
```
- Serialized into a compact payload with only 33 bytes compared to the 95 bytes required for JSON
- View serialization breakdown by running `imgcat assets/protobuf-serialization.png`

---

## When to use gPRC and REST

Use REST if:
- You are building generic web APIs to be consumed by anyone
    - REST will be more easily adopted and will have more tooling around it
- You need browser compatibility
    - gRPC uses HTTP/2 features and you won't find many browsers that can support it as of now

Use gRPC if:
- You are building APIs to be consumed by internal microservices
- You need really high performance and low latency
    - HTTP/2 and protobuf will make gRPC much more performant than REST
- Real-time data streaming, especially when bidirectional streaming is required

---

## Considerations When Using gRPC

### Schema Evolution

Schemas change over time. When updating your schemas, you'll need to ensure you're not breaking either backwards or forwards compatibility.
- Backwards compatibility means new code needs to be able to handle reading data written by old code
    - a server using a new `.proto` definition should still be able to handle requests from clients which haven't updated to the latest versions
    - the server will not require new fields to be present, and will ignore the values passed in for any deprecated fields
- Forwards compatibility means old code needs to be able to handle reading data written by new code
    - a client using an old `.proto` definition should be able to handle reading any response from a server on a newer version
    - client will ignore newly added fields in the response

Below is the previous `.proto` definition with valid updates:
```proto
message PersonRequest {
    string user               = 1;                   // field names can change because the name does not matter for encoding, but the type and tag must not change
    int64 favorite_number     = 2 [deprecated=true]; // deprecated fields are not removed to ensure backwards compatibility
    repeated string interests = 3;
    repeated string friends   = 4;                   // newly added fields use new tag numbers, old numbers must never be re-used
}

message PersonResponse {}

service PersonService {
    rpc AddPerson(PersonRequest) returns (PersonResponse) {}
}
```
All fields in the `.proto` file are implicitly `optional`. We can annotate them as `required`, but that makes evolving the schema and not breaking things a lot harder.

---

## Considerations When Using gRPC

### Schema Maintenance and Versioning
Something that needs to be decided on when using gRPC in your projects is _where_ the source of truth to your .proto definitions will actually be

1. Monorepo
- Really good choice if you intentionally make that decision upfront
- All your clients and servers would live in the same repository and can easily reference changes in the schema
- A schema update could be rolled out in a single PR

2. Dedicated schema repository
- Allows for a centralized location for all your schema management
- A schema update would require at least (2 + _n_) PRs where _n_ is the number of client repositories

3. A schema registry like Buf
- Makes it easier to document your `.proto` definitions
- Allows for easy schema versioning
- A schema update would require at least (1 + _n_) PRs where _n_ is the number of client repositories

---

## A tour of Buf

Buf provides tooling that makes gRPC a lot easier to work with, including the Buf CLI, and the Buf Schema Registry. We'll be able to take a look at these features in the upcoming demo.

### Buf CLI
- Lint, format, and detect breaking changes in protobuf files
- Generate code stubs for multiple languages
- Manage dependencies on other protobuf files
- Push proto definitions to the Buf Schema Registry

### Buf Schema Registry
- A registry to host all your `.proto` definitions
- Supports public and private registries

[buf.build](https://buf.build/)


---


## Demo
- Run a gRPC server and client, explore the existing API
- Modify the `.proto` definitions
- Push the new schema up to the Buf Schema Registry
- Update the server and clients to support the new schema
- Re-run the gRPC server and clients
- The apps are going to be deployed on a local k8s cluster, because we work at Spectro Cloud :)

Code can be found in the following repos
- [count-server](https://github.com/ahmad-ibra/count-server)
- [count-client](https://github.com/ahmad-ibra/count-client)
- [count-k8s](https://github.com/ahmad-ibra/count-k8s)

---

## Buf at Spectro Cloud
spectro-cleanup is a generic cleanup utility for removing arbitrary files from nodes and/or resources from a K8s cluster.
- Currently we're using the Buf Schema Registry to host `.proto` definitions for spectro-cleanup. 
- https://github.com/spectrocloud-labs/spectro-cleanup/blob/main/proto/cleanup/v1/cleanup.proto

validator is using spectro-cleanup to clean up our validator-plugins. It makes a gRPC request to spectro-cleanup notifying it that its ready to have everything removed.
- https://github.com/spectrocloud-labs/validator/blob/cdb6424ce504ce1bdb347f92f8cc0b453f857096/internal/controller/validatorconfig_controller.go#L397

---

## Thank you for coming to my TED talk!
## Questions?
