# OpenFaaS gRPC Ingress Controller Controller

yeah, it's a double controller.

to put it quickly, this searches for any pods under openfaas-fn (the ones it creates,) and searches for an annotation, `com.roleypoly/faas-rpc` which is set to the "gRPC path" of the function underneath.

this controller creates **FunctionIngress** objects matching what gRPC native and improbable-eng/gRPC-web systems are expecting.

the overall goal was to have *individual functions* in a gRPC system exist as a function alone. this part solves routing.

## Deploying

clone this repo,
```sh
kubectl apply -f ./k8s
```

## Example

this example implies you are running a recent OpenFaaS install with nginx-ingress-controller and OpenFaaS's ingress-operator **1.5.0 or newer**.

consider this well-known RPC definition, **greeter.proto**...

```proto
package greeter; // Note this, if it's missing, omit it from instructions.

// The greeting service definition.
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
```

your **stack.yaml** would look like this to accomodate it

```yaml
version: 1.0
provider:
  name: openfaas
functions:
  greeter-say-hello:
    lang: golang-grpc # this is the other piece of the puzzle! see below.
    handler: ./greeter-say-hello
    image: mydocker/greeter-say-hello:latest
    annotations:
      com.roleypoly/faas-rpc: greeter.Greeter/SayHello # *** omit `greeter.` if `package` is not set in protobuf.
```

and it should just work given this controller is deployed.

### wait a minute

[**golang-grpc**](https://github.com/roleypoly/faas-templates) is the other piece of the puzzle. it provides the function wrapper pattern to support this working.

see [example repo](https://github.com/roleypoly/faas-rpc-example) for an end-to-end example that you can run on your friendliest kubernetes cluster.

## todo

- [x] it works
- [ ] other ingresses (nginx only right now, no real reason)
- [ ] example
- [ ] verify on raspi (arm/v7 + arm64 images exist, though.)