# Overview of Source Controller

Tanzu Source Controller provides a standard interface for artifact acquisition and extends the
function of [Flux CD Source Controller](../fluxcd-source-controller/about.hbs.md).

Tanzu Source Controller supports the following two resource types:

- ImageRepository [Deprecated]
- MavenArtifact

An `ImageRepository` resource can resolve the source from the contents of an image in an image registry.
This enables app developers to create and update workloads from local source code or a code repository.

A `MavenArtifact` resource can resolve a binary artifact from a Maven repository. This functionality enables the supply chain to support artifacts produced externally.

>**Note** Fetching `RELEASE` version from GitHub packages is not currently supported. The `metadata.xml` in GitHub packages does not have the `release` tag that contains the released version number. For more information, see [Maven-metadata.xml is corrupted on upload to registry](https://github.community/t/maven-metadata-xml-is-corrupted-on-upload-to-registry/177725) on GitHub.
