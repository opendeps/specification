# The Open Dependencies project (OpenDeps)

OpenDeps allows you to express your application's _external runtime dependencies_. Using OpenDeps, you use a YAML file to clearly communicate what APIs your software component needs to run correctly.

Formalising your application's runtime dependencies helps improve software quality (e.g. earlier integration testing) and operations (e.g. deployment automation) to help you move faster, safely.

Benefits:
- Verify dependencies are in place before deploying your app
- Rapidly spin up live mocks of all your app's dependencies so you can build/test your app quickly
- Integrates with the [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification)
- Language agnostic and cross-platform
- Human readable and machine readable; comprehensible by developers and non-developers

Here is an example:

```yaml
dependencies:
  order_service:
    summary: The order service.
    spec: https://raw.githubusercontent.com/opendeps/specification/main/examples/petstore/order_service_openapi.yaml
    version: "2.0.0"
    availability:
      url: https://example.com/orders/healthz
      security: basicAuth

  pet_name_service:
    summary: The pet naming service.
    spec: ./pet_name_service_openapi.yaml
    version: "1.0.0"
```

Some things to note about this example:

- Our Petstore app depends on two external services: `order_service` and `pet_name_service`. Each is defined by an [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification).
- The services expose an availability URL that responds with an `HTTP 2xx` status to indicate it is up. This lets us probe for the existence of these dependencies, such at deploy time or for operational health monitoring.
- Our app requires version `2.0.0` of the `order_service` to be present and version `1.0.0` of `pet_name_service`. Tools check the OpenDeps version requirement against the OpenAPI spec.

> See [a more complete example](./docs/detailed-example.md) for more details about use and options.

## How is this different from npm/Maven/Gradle/Go Modules etc.?

OpenDeps is for expressing your _external runtime dependencies_, rather than things needed to build or package your application.

Applications often depend on external APIs and other resources. It's common to need specific versions of these APIs for your software to function correctly.

During development OpenDeps improves the speed of the 'inner loop' of test-build on your local machine. Tools create live working mock endpoints of your API dependencies, based on their OpenAPI specs.

Ship an OpenDeps specification alongside your application, so at deployment time you can verify that all required dependencies exist. This can save you deploying an application into an environment in which it cannot work, saving your users from service outages.

## Use case 1: Local development

You are building your application and would like to test it locally. Like many applications, yours depends on third party APIs for various services. With a single command, all your application's external dependencies are running, in mock form, on your local machine.

With its dependencies running, you can now interactively test your application against local mocks, saving you deployment effort and providing a much faster feedback loop.

This is possible because tools like [Imposter](https://github.com/outofcoffee/imposter/) can understand your app's dependencies, fetch their OpenAPI specifications, and automatically create live mocks of those dependencies.

You can mark which dependencies are required in your OpenDeps specification:

```yaml
dependencies:
  order_service:
    summary: The order service.
    description: The service allows ordering of pet supplies from the store.
    spec: https://raw.githubusercontent.com/opendeps/specification/main/examples/petstore/order_service_openapi.yaml
    version: "2.0.0"
    required: true

  pet_name_service:
    summary: The pet naming service.
    spec: ./pet_name_service_openapi.yaml
    version: "1.0.0"
    required: false
```

Dependencies not marked as `required` ('optional dependencies') represent those without which your application can still operate, but perhaps in a degraded mode.

## Use case 2: Deployment

You have built and tested your application and it has reached the deployment stage of your CI/CD pipeline. Just before you trigger your deployment operation, your tools spot that one of the application's _required_ dependencies is not available.

Perhaps another team has not yet deployed the version you require, or maybe the third party endpoint is not reachable from your application's environment. Thanks to the availability probe, your deployment can fail fast, avoiding causing a service outage for your users.

This is possible because tools verify dependency availability using the `availability.url` value. Those marked with `required: true` must be available for your application to work properly.

### Availability probe configuration

You can customise the values used to check for availability, or use the defaults. If your availability endpoint requires it, you can customise the security configuration.

> Learn more about [availability probes](./docs/availability-probe.md).

---

## The specification

The [OpenDeps specification](./opendeps-specification.json) is defined in JSON Schema.

### Examples

See the `examples` directory for sample OpenDeps files, including:

- [Petstore - multiple dependencies](./examples/petstore/opendeps.yaml)
- [Authentication - various types](./examples/authentication/opendeps.yaml)

### Documentation

- [Detailed example](./docs/detailed-example.md)
- [Availability probes](./docs/availability-probe.md)
- [Self-hosting OpenDeps files](./docs/self-hosting.md)
- [Metadata](./docs/metadata.md)

---

## The OpenDeps CLI

The OpenDeps command line tool provides the following features:

* Start live mocks of dependencies
* Test the availability of dependencies
* Validate OpenDeps files against this specification

It's open source and available cross-platform.

> [Get the OpenDeps CLI](https://github.com/opendeps/cli) for your system.

---

## Contributing

Suggestions and improvements to the specification, examples, or documentation are welcome. Please raise them as GitHub issues.
