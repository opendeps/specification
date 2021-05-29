# The Open Dependencies project (OpenDeps)

The OpenDeps specification allows you to express your application's _external_ runtime dependencies. This allows you to clearly communicate the required versions of your dependencies. This has many benefits in software quality and deployment automation to help you move faster, safely.

Benefits:
- Human readable and machine readable; language agnostic
- Tooling can verify your dependencies are in place before deploying your app
- Rapidly spin up live mocks of all your app's dependencies so you can build/test your app quickly
- Integrates with the [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification)
- Comprehensible by developers and non-developers

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

In this example our app, a Petstore, depends on two external services. Each service's specification is a regular [OpenAPI specification](https://github.com/OAI/OpenAPI-Specification).

For example, our app requires version `2.0.0` of the `pet_name_service` to be present. The `pet_name_service` has an availability URL that should respond with `HTTP 200 OK` to indicate it is available.

Our app also requires version `1.0.0` of `order_service`. This dependency is marked as `required: true` which means that it being unreachable must be considered a failure. I

### Metadata

Dependencies can have metadata to ensure a more specific match to your app's requirements.

The types of metadata are as follows:

- OpenAPI spec: each service can provide an API specification using the OpenAPI standard. This provides information about the dependency, such as its version and endpoints ('servers' in OpenAPI 3.x).
- 

## Why is this different from npm/Maven/Gradle/godep etc.?

OpenDeps is for expressing your external runtime dependencies, rather than things needed to build or package your application. An OpenDeps specification is intended to be shipped alongside your application, so that at deployment time, you can verify that your required dependencies exist. This can save you from deploying your application into an environment in which it will not work, saving your users from service outages.

Another benefit OpenDeps provides is during local development - rapidly improving the speed of the 'inner loop' of test-build on your local enviroment. Tools understand that your application needs certain APIs to work, so they can stand up live working mock endpoints of these APIs based on their OpenAPI specs.

## Use case 1: Local development

You are building your application and would like to test it. With a command, all your application's dependencies are running, in mock form, on your local machine.

With your app's dependencies running, you can now interactively test your application against local mocks, saving you deployment effort and providing a much faster feedback loop.

This is possible because tools like [Imposter](https://github.com/outofcoffee/imposter/) can understand your app's dependencies and download their OpenAPI specifications to automatically create live mocks of those dependencies.

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

You have built and tested your application and it has reached the deployment stage of your CI/CD pipeline. Just before you trigger your deployment operation, your tools spot that one of the application's _required_ dependencies is not yet deployed.

Perhaps your dependency hasn't been deployed yet, or is the wrong version, or maybe it is not reachable from your application's environment. You can choose to fail fast to avoid causing a service outage for your users.

This was possible because tooling can parse the dependencies marked with `required: true` and verify their availability using the `availability.url` value.

You can customise the values used to check for availability, or use the defaults:

```yaml
availability:
  url: https://example.com/orders/healthz
  # Check every 10 seconds, trying 3 times and waiting 5 seconds each time
  intervalSeconds: 10
  timeoutSeconds: 5
  attempts: 3
```

You can customise the security requirements for accessing the availability URL:

```yaml
availability:
  url: https://example.com/orders
  security:
    - BasicAuth: []
```

The security schemes are defined in the `components.securitySchemes` block:

```yaml
components:
  securitySchemes:
    BasicAuth:
      type: http
      scheme: basic

```

## Self-hosting OpenDeps files in your app

In trusted environments, you may wish to serve your OpenDeps manifest from your application.

**Important:** You should consider who should have access to the information in your OpenDeps manifest, as it may contain sensitive data about your appliction.

By convention, you host your application's OpenDeps file at the following path:

    /.well-known/opendeps/manifest.yaml

For example:

    https://example.com/petstore/.well-known/opendeps/manifest.yaml

Tools use this well known path to obtain the OpenDeps manifest for your app, enabling automation of operations tasks such as monitoring.

## Contributing

Suggestions and improvements to the specification, examples, or documentation are welcome. Please raise them as GitHub issues.
