# The Open Dependencies project (OpenDeps)

OpenDeps allows you to express your application's _external runtime dependencies_. Clearly communicate what your software component needs to operate and run correctly.

Your application requires specific versions of external APIs and other resources. Formalising these dependencies helps improve software quality (e.g. earlier integration testing) and operations (e.g. deployment automation) to help you move faster, safely.

Benefits:
- Verify dependencies are in place before deploying your app
- Rapidly spin up live mocks of all your app's dependencies so you can build/test your app quickly
- Integrates with the [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification)
- Language agnostic and cross-platform
- Human readable and machine readable; comprehensible by developers and non-developers

Here is an example:

```yaml
opendeps: "0.1.0"

info:
  title: Sample Petstore app dependencies
  description: This is a sample specification declaring the dependencies of an example Petstore app.
  version: "1.0.1"

dependencies:
  pet_name_service:
    summary: The pet naming service.
    description: The service generates pet names.
    spec: ./pet_name_service_openapi.yaml
    version: "2.0.0"
    availability:
      url: https://example.com/pet-names/healthz

  order_service:
    summary: The order service.
    description: The service allows ordering of pet supplies from the store.
    spec: https://raw.githubusercontent.com/opendeps/specification/main/examples/petstore/order_service_openapi.yaml
    version: "1.0.0"
    required: true
    availability:
      url: https://example.com/orders/healthz
      intervalSeconds: 10
      timeoutSeconds: 5
      attempts: 3
```

Some things to note about this example:

- Our Petstore app depends on two external services: `pet_name_service` and `order_service`. Each service is defined by a regular [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification).
- The services expose an *availability URL* that must respond with an `HTTP 2xx` status to indicate they are up. This lets us test for the successful existence of these dependencies, such at deploy time or for operational health monitoring.
- Our app requires version `2.0.0` of the `pet_name_service` to be present and version `1.0.0` of `order_service`. Tools can validate the OpenDeps requirement against the OpenAPI spec.
- The `order_service` dependency is marked as `required: true` which means that it being unreachable is considered a failure, unlike the `pet_name_service`.

## Why is this different from npm/Maven/Gradle/godep etc.?

OpenDeps is for expressing your external runtime dependencies, rather than things needed to build or package your application.

You can ship an OpenDeps specification alongside your application, so that at deployment time, you can verify that your required dependencies exist. This can save you from deploying your application into an environment in which it will not work, saving your users from service outages.

During development OpenDeps helps you improve the speed of the 'inner loop' of test-build on your local environment. Tools can stand up live working mock endpoints of your API dependencies based on their OpenAPI specs, because OpenDeps expresses what your application needs to run properly.

## Use case 1: Local development

You are building your application and would like to test it. Like many applications, yours depends on third party APIs for various services. With a single command, all your application's dependencies are running, in mock form, on your local machine.

With your app's dependencies running, you can now interactively test your application against local mocks, saving you deployment effort and providing a much faster feedback loop.

This is possible because tools like [Imposter](https://github.com/outofcoffee/imposter/) can understand your app's dependencies, fetch their OpenAPI specifications, and automatically create live mocks of those dependencies.

You can mark which dependencies are required in your OpenDeps specification:

```yaml
dependencies:
  pet_name_service:
    summary: The pet naming service.
    spec: ./pet_name_service_openapi.yaml
    version: "2.0.0"
    required: false

  order_service:
    summary: The order service.
    description: The service allows ordering of pet supplies from the store.
    spec: https://raw.githubusercontent.com/opendeps/specification/main/examples/petstore/order_service_openapi.yaml
    version: "1.0.0"
    required: true
```

Non-required dependencies ('optional dependencies') are those without which your application can still operate, but perhaps in a degraded mode.

## Use case 2: Deployment

You have built and tested your application and it has reached the deployment stage of your CI/CD pipeline. Just before you trigger your deployment operation, your tools spot that one of the application's _required_ dependencies is not available.

Perhaps another team has not deployed the version you require yet, or maybe the third party endpoint is not reachable from your application's environment. Your deployment can fail fast, avoiding causing a service outage for your users.

This is possible because tooling parse the dependencies marked with `required: true` and verify their availability using the `availability.url` value.

You can customise the values used to check for availability, or use the defaults:

```yaml
availability:
  url: https://example.com/orders/healthz
  # Check every 10 seconds, trying 3 times and waiting 5 seconds each time
  intervalSeconds: 10
  timeoutSeconds: 5
  attempts: 3
```

### Availability probe security

If your availability endpoint requires it, you can customise the security configuration:

```yaml
availability:
  url: https://example.com/orders
  security: basicAuth
```

The security configurations are defined in the `components.securityConfigs` block:

```yaml
components:
  securityConfigs:
    basicAuth:
      type: authHeader
      scheme: basic
```

> This example describes the following HTTP request header:
> ```Authorization: Basic base64(<username:password>)```

Other supported security configurations include Bearer token and custom HTTP headers.

You do not store secrets or tokens within the OpenDeps file - these are provided by the tooling.

---

## The specification

The [OpenDeps specification](./opendeps-specification.json) is defined in JSON Schema.

### Examples

See the `examples` directory for sample OpenDeps files, including:

- [Petstore - multiple dependencies](./examples/petstore/opendeps.yaml)
- [Authentication - various types](./examples/authentication/opendeps.yaml)

---

## Self-hosting OpenDeps files in your app

In trusted environments, you may wish to serve your OpenDeps manifest from your application.

**Important:** You should consider who should have access to the information in your OpenDeps manifest, as it may contain sensitive data about your appliction.

By convention, you host your application's OpenDeps file at the following path:

    /.well-known/opendeps/manifest.yaml

For example:

    https://example.com/petstore/.well-known/opendeps/manifest.yaml

Tools use this well known path to obtain the OpenDeps manifest for your app, enabling automation of operations tasks such as monitoring.

### Metadata

Dependencies have metadata to ensure a more specific match to your app's requirements.

The types of metadata are as follows:

- OpenAPI spec: each dependency can provide an API specification using the OpenAPI standard. This provides information about the dependency, such as its version and endpoints ('servers' in OpenAPI 3.x).
- Inline metadata: each dependency can specify the required version, which can be validated against its specification, to ensure that the right version is deployed

## Contributing

Suggestions and improvements to the specification, examples, or documentation are welcome. Please raise them as GitHub issues.
