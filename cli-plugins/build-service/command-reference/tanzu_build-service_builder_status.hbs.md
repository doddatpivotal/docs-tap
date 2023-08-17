# tanzu build-service builder status

This topic tells you how to use the Tanzu Build Service CLI `tanzu build-service builder status` command
to display the status of a builder.

## Synopsis

Prints detailed information about the status of a specific builder in the provided namespace.

The namespace defaults to the kubernetes current-context namespace.

```console
tanzu build-service builder status <name> [flags]
```

## Examples

```console
tanzu build-service builder status my-builder
tanzu build-service builder status -n my-namespace other-builder
```

## Options

```console
  -h, --help               help for status
  -n, --namespace string   kubernetes namespace
```

## SEE ALSO

* [tanzu build-service builder](tanzu_build-service_builder.hbs.md)	 - Builder Commands
