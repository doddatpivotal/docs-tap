# tanzu build-service build list

This topic tells you how to use the Tanzu Build Service CLI `tanzu build-service build list` command
to lists builds.

## Synopsis

Prints a table of the most important information about builds in the provided namespace.

The namespace defaults to the Kubernetes current-context namespace.

```console
tanzu build-service build list [image-resource-name] [flags]
```

## Examples

```console
tanzu build-service build list
tanzu build-service build list my-image
tanzu build-service build list my-image -n my-namespace
```

## Options

```console
  -h, --help               help for list
  -n, --namespace string   kubernetes namespace
```

## SEE ALSO

* [tanzu build-service build](tanzu_build-service_build.hbs.md)	 - Build Commands
