# Detailed example

This section describes a more complete OpenDeps document.

> Also see the `examples` directory for sample OpenDeps files, including:
>
> - [Petstore - multiple dependencies](./examples/petstore/opendeps.yaml)
> - [Authentication - various types](./examples/authentication/opendeps.yaml)

## Specification

```yaml
opendeps: "0.1.0"

info:
  title: Sample Petstore app
  description: Declares the dependencies of an example Petstore app.
  version: "1.0.1"

dependencies:
  # We depend on version 2.0.0 of the order service.
  # In development, we get an instant live mock of the service using its OpenAPI spec.
  # At deployment time, we check it is available before deploying our app.
  order_service:
    summary: The order service.
    description: The service allows ordering of pet supplies from the store.
    # The OpenAPI specification for the order service.
    spec: https://raw.githubusercontent.com/opendeps/specification/main/examples/petstore/order_service_openapi.yaml
    # This version must be deployed for our app to work.
    version: "2.0.0"
    # It's a required dependency at runtime.
    required: true
    # How to probe order service availability.
    availability:
      url: https://example.com/orders/healthz
      security: basicAuth

  # We also use the pet name service, but it is not essential,
  # so it's not marked as 'required' here.
  #
  # We can still get a live mock of the service during development/testing,
  # but if the service is not available at deployment time, just log a warning.
  pet_name_service:
    summary: The pet names service.
    description: The service generates pet names.
    # Our OpenAPI spec is relative to the OpenDeps file.
    spec: ./pet_name_service_openapi.yaml
    version: "1.0.0"
    # This block controls how we probe to see if the service is up.
    availability:
      url: https://example.com/pet-names/healthz
      intervalSeconds: 10
      timeoutSeconds: 5
      attempts: 3

components:
  securityConfigs:
    # HTTP basic authentication
    basicAuth:
      type: authHeader
      scheme: basic
```

Some things to note about this example:

- Our Petstore app depends on two external services: `order_service` and `pet_name_service`. Each is defined by an [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification).
- The services expose an availability URL that responds with an `HTTP 2xx` status to indicate it is up. This lets us probe for the existence of these dependencies, such at deploy time or for operational health monitoring.
- Our app requires version `2.0.0` of the `order_service` to be present and version `1.0.0` of `pet_name_service`. Tools check the OpenDeps version requirement against the OpenAPI spec.
- The `order_service` dependency is marked as `required: true` which means that it being unreachable is considered a failure, unlike the `pet_name_service`.
