# Availability probes

Tools verify dependency availability using the `availability.url` value. Those marked with `required: true` must be available for your application to work properly.

### Availability probe configuration

You can customise the values used to check for availability, or use the defaults:

```yaml
availability:
  # This is the endpoint to probe.
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
  url: https://example.com/orders/healthz
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
> ```Authorization: Basic <base64 encoded username:password>```

Other supported security configurations include Bearer token and custom HTTP headers.

Secrets or tokens are not stored within an OpenDeps file - these are provided by the tooling.

