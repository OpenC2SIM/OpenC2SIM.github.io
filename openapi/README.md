# C2SIM Server REST Interface — OpenAPI

This directory contains an OpenAPI 3.0.3 description of the C2SIM server REST
interface and a ready-to-use generator configuration, so client and server
stubs no longer need to be written by hand (see issues #2 and #3).

## Files

- `c2sim-server.yaml` — the interface description, drafted from *C2SIM Server
  Reference Implementation Documentation 4.8.3.1*. It covers
  `POST /C2SIMServer/c2sim` (BML and C2SIM document submission) and
  `POST /C2SIMServer/command` (session commands). The separate STOMP interface
  is out of scope.
- `config-python.yaml` — OpenAPI Generator configuration for a Python client
  (`packageName: c2sim_client`, `projectName: c2sim-client`).

Generated code is **not** committed; produce it locally as needed.

## Generate a client

With Java 17+ (no Docker required):

```bash
curl -L -o openapi-generator-cli.jar \
  https://repo1.maven.org/maven2/org/openapitools/openapi-generator-cli/7.10.0/openapi-generator-cli-7.10.0.jar

java -jar openapi-generator-cli.jar generate \
  -g python \
  -i openapi/c2sim-server.yaml \
  -o out/python \
  -c openapi/config-python.yaml
```

Or with Docker:

```bash
docker run --rm -v "${PWD}:/local" openapitools/openapi-generator-cli generate \
  -g python \
  -i /local/openapi/c2sim-server.yaml \
  -o /local/out/python \
  -c /local/openapi/config-python.yaml
```

Other generators (`java`, `spring`, `typescript`, `go`, …) work the same way;
only `-g` and the config file need to change.

## Verified example

The commands above were run with OpenAPI Generator 7.10.0. The generated
Python package imports cleanly and exposes
`c2_sim_server_c2sim_post` and `c2_sim_server_command_post`.

A request made with that client against a local mock HTTP server arrived as:

```
POST /C2SIMServer/c2sim?protocol=C2SIM&submitterID=tester&version=4.8.3.1
```

with the submitted XML document (`<C2SIMInitialization/>`) intact in the body,
and the mock's response returned to the caller. The spec itself also passes
`openapi-spec-validator`.

## Caveats

- The reference documentation lists the inputs as "REST Parameters" without
  specifying their transport, so they are modelled as **query parameters**. If
  the reference implementation expects `application/x-www-form-urlencoded`
  bodies instead, adjust `c2sim-server.yaml` before generating code.
- Each call is a single REST transaction; the connection does not persist.
  Generated clients may offer keep-alive/session features that are irrelevant
  here.
- Verification used a mock server only. Exercise the generated client against
  a real C2SIM server before relying on it in an exercise or coalition.
