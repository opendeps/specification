## Self-hosting OpenDeps files in your app

In trusted environments, you may wish to serve your OpenDeps manifest from your application.

**Important:** You should consider who should have access to the information in your OpenDeps manifest, as it may contain sensitive data about your application.

By convention, you host your application's OpenDeps file at the following path:

    /.well-known/opendeps/manifest.yaml

For example:

    https://example.com/.well-known/opendeps/manifest.yaml

Tools use this well known path to obtain the OpenDeps file for your app, enabling automation of operations tasks such as monitoring.
