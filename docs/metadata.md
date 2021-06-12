# Types of metadata

This section describes the types of metadata in an OpenDeps document.

Dependencies have metadata. The more specific the metadata, the greater the confidence an environment meets your app's requirements.

Key metadata:

- **OpenAPI spec**: each dependency can provide an API specification using the OpenAPI standard. This provides information about the dependency, such as its version and endpoints.
- **Inline metadata**: each dependency can specify the required version, which can be validated against its specification, to ensure that the right dependency is present

It is good practice to specify the version of each dependency, the URL to its specification and its availability probe endpoint. This enables tools to automate more development and deployment tasks.
