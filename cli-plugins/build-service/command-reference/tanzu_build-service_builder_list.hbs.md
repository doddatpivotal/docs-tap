# tanzu build-service builder list

This topic tells you how to use the Tanzu Build Service CLI `tanzu build-service build list` command
to list available builders.

## Synopsis

Prints a table of the most important information about the available builders in the provided namespace.

The namespace defaults to the kubernetes current-context namespace.

```console
tanzu build-service builder list [flags]
```

## Examples

```console
tanzu build-service builder list
tanzu build-service builder list -n my-namespace
```

## Options

```console
  -h, --help               help for list
  -n, --namespace string   kubernetes namespace
```

## SEE ALSO

* [tanzu build-service builder](tanzu_build-service_builder.hbs.md)	 - Builder Commands
