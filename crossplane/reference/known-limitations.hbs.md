# Crossplane limitations

This topic tells you about the limitations related to Crossplane on Tanzu Application Platform
(commonly known as TAP).

For troubleshooting guidance, see [Troubleshoot Crossplane](../how-to-guides/troubleshooting.hbs.md).

## <a id="too-many-crds"></a> Cluster performance degradation due to large number of CRDs

Take care before installing extra Crossplane Providers into Tanzu Application Platform.
Some Providers install hundreds of additional CRDs into the cluster.

This is particularly true of the Providers for AWS, Azure, and GCP.
For the number of CRDs installed with these Providers, see:

- [provider-aws CRDs](https://marketplace.upbound.io/providers/upbound/provider-aws/latest/managed-resources)
- [provider-azure CRDs](https://marketplace.upbound.io/providers/upbound/provider-azure/latest/managed-resources)
- [provider-gcp CRDs](https://marketplace.upbound.io/providers/upbound/provider-gcp/latest/managed-resources)

You must ensure that your cluster has sufficient resource to support this number of additional CRDs
if you choose to install them.

**Workaround:**

To address this issue, Upbound have released a new feature named Provider Families.
For more information, see the [Crossplane blog](https://blog.crossplane.io/crd-scaling-provider-families/).
