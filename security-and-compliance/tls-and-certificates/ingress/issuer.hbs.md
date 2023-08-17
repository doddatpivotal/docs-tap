# Shared Ingress issuer in Tanzu Application Platform

This topic tells you about the Tanzu Application Platform (commonly known as TAP) shared ingress issuer.

The shared ingress issuer is an on-platform representation of a certificate authority.
It provides a method to set up TLS for the entire platform. All participating components get their ingress certificates issued by it.

This is the recommended best practice for issuing ingress certificates on Tanzu
Application Platform. Learn about its [prerequisites](#prerequisites).

The ingress issuer is designated by the single Tanzu Application Platform configuration value
`shared.ingress_issuer`. It refers to a `cert-manager.io/v1/ClusterIssuer`.

By default, a self-signed issuer is used. It's called `tap-ingress-selfsigned`
and has limitations. For more information, see [Limitations of the default, self-signed issuer](#limitations-self-signed).

VMware recommends that you replace the default self-signed issuer with your own issuer.
For more information, see [Replacing the default ingress issuer](#replace).

Component-level configuration of TLS takes precedence and can be
mixed with the ingress issuer. For more information, see [Overriding TLS for components](#override).

You can deactivate the ingress issuer. For more information, see
[Deactivating TLS for ingress](#deactivate).

## <a id="prerequisites"></a>Prerequisites

To use the Tanzu Application Platform ingress issuer, your _certificate authority_ must be
representable by a cert-manager `ClusterIssuer`. In particular, you need one of
the following:

- Your own CA certificate
- Your CA is an ACME, Venafi, or Vault-based issuer, for example, _LetsEncrypt_
- Your CA can be represented by an
  [external](https://cert-manager.io/docs/configuration/external/) cert-manager
  `ClusterIssuer`.

Without one of the above, you cannot use the issuer ingress, but you can
still configure TLS for components. For more information, see
[Ingress certificates inventory](./inventory.hbs.md).

## <a id="prerequisites"></a>Default

By default, Tanzu Application Platform installs and uses a self-signed CA as
its ingress issuer for all components.

This default ingress issuer is a self-signed `cert-manager.io/v1/ClusterIssuer`
and is provided by Tanzu Application Platform's [cert-manager
package](../../../cert-manager/about.hbs.md). Its default name is
`tap-ingress-selfsigned`.

The default ingress issuer is appropriate for testing and evaluation, but VMware recommends that you
replace it with your own issuer.

>**Important** If `cert-manager.tanzu.vmware.com` is excluded from the
>installation, then `tap-ingress-selfsigned` is not installed either. In
>this case, bring your own ingress issuer.

### <a id="limitations-self-signed"></a>Limitations of the default, self-signed issuer

The default ingress issuer represents a self-signed _certificate authority_.
This is not problematic as far as security is concerned, however, it
is not included in any trust chain configured.

As a result, nothing trusts the default ingress issuer implicitly, not even
Tanzu Application Platform components. While the issued certificates are valid in principal, they
are rejected, for example, by your browser. Furthermore, some interactions between components are
not functional by default.

### <a id="trust-self-signed"></a>Trusting the default, self-signed issuer

You can trust the default ingress issuer by including
`tap-ingress-selfsigned`'s certificate in the Tanzu Application Platform trusted CA certificates and your device's certificate chain.

> **Caution** This approach is discouraged. Instead, replace the default ingress issuer.

1. Obtain `tap-ingress-selfsigned`'s PEM-encoded certificate

  ```shell
  kubectl get secret \
    tap-ingress-selfsigned-root-ca \
    --namespace cert-manager \
    --output go-template='\{{ index .data "tls.crt" | base64decode }}'
  ```

1. Add the certificate to [custom CA
   certificates](../custom-ca-certificates.hbs.md) by appending it to
   `shared.ca_cert_data` and applying Tanzu Application Platform's installation values

1. Add the certificate to your device's trust chain. The trust chain will vary depending on your
   operating system and privileges.

## <a id="replace"></a>Replacing the default ingress issuer

Tanzu Application Platform's default ingress issuer can be replaced by any
other [cert-manager-compliant
ClusterIssuer](https://cert-manager.io/docs/configuration/).

To replace the default ingress issuer:

Custom CA
: Complete the following steps:

  **Prerequisites**

  You need your own CA certificates and private key for this.

  1. Create your `ClusterIssuer`

      Create a `Secret` and `ClusterIssuer` which represent your CA on the platform:

      ```yaml
      ---
      apiVersion: v1
      kind: Secret
      type: kubernetes.io/tls
      metadata:
        name: my-company-ca
        namespace: cert-manager
      stringData:
        tls.crt: #! your CA's PEM-encoded certificate
        tls.key: #! your CA's PEM-encoded private key

      ---
      apiVersion: cert-manager.io/v1
      kind: ClusterIssuer
      metadata:
        name: my-company
      spec:
        ca:
          secretName: my-company-ca
      ```

  1. Set `shared.ingress_issuer` to the name of your issuer:

      ```yaml
      #! my-tap-values.yaml
      #! ...
      shared:
        ingress_issuer: my-company-ca
      #! ...
      ```

  1. Apply the Tanzu Application Platform installation values file.

      When the configuration is applied, components obtain certificates
      from the new issuer and serve them.

LetsEncrypt production
: Complete the following steps

  **Prerequisites**

  - Public CAs, like LetsEncrypt, record signed certificates in
   publicly-available certificate logs for the purpose of [certificate
   transparency](https://certificate.transparency.dev/). Ensure that you are
   OK with this before using LetsEncrypt.
  - LetsEncrypt's production API has [rate
   limits](https://letsencrypt.org/docs/rate-limits/).
  - LetsEncrypt requires your `shared.ingress_domain` to be accessible from
   the Internet.
  - Depending on your setup, you might need to adjust
   [.spec.acme.solvers](https://cert-manager.io/docs/configuration/acme/#solving-challenges)
  - Replace `.spec.acme.email` with the email which should receive notices
   for certificates from LetsEncrypt.

  > **Caution** ACME HTTP01 challenges can fail under certain conditions. 
  > For more information, see [ACME challenges](../../../cert-manager/acme-challenges.hbs.md).

  1. Create a `ClusterIssuer` for [Let's Encrypts](https://letsencrypt.org)
     production API:

      ```yaml
      ---
      apiVersion: cert-manager.io/v1
      kind: ClusterIssuer
      metadata:
        name: letsencrypt-production
      spec:
        acme:
          email: certificate-notices@my-company.com
          privateKeySecretRef:
            name: letsencrypt-production
          server: https://acme-v02.api.letsencrypt.org/directory
          solvers:
            - http01:
                ingress:
                  class: contour
      ```

  1. Set `shared.ingress_issuer` to the name of your issuer

      ```yaml
      #! my-tap-values.yaml
      #! ...
      shared:
        ingress_issuer: letsencrypt-production
      #! ...
      ```

  1. Apply Tanzu Application Platform installation values

      When the configuration is applied, components  obtain certificates
      from the new issuer and serve them.

LetsEncrypt staging
: Complete the following steps

  **Prerequisites**

  - Public CAs - like LetsEncrypt - record signed certificates in
    publicly-available certificate logs for the purpose of [certificate
    transparency](https://certificate.transparency.dev/). Ensure that you are
    OK with this before using LetsEncrypt.
  - LetsEncrypt's staging API is not a publicly-trusted CA. You have
    to add its certificate to your devices trust chain and [Tanzu Application Platform's custom CA
    certificates](../custom-ca-certificates.hbs.md).
  - LetsEncrypt requires your `shared.ingress_domain` to be accessible from
    the Internet.
  - Depending on your setup you might need to adjust
    [.spec.acme.solvers](https://cert-manager.io/docs/configuration/acme/#solving-challenges).
  - Replace `.spec.acme.email` with the email which should receive notices
    for certificates from LetsEncrypt.

  > **Caution** ACME HTTP01 challenges can fail under certain conditions. 
  > For more information, see [ACME challenges](../../../cert-manager/acme-challenges.hbs.md).

  1. Create a `ClusterIssuer` for [Let's Encrypts](https://letsencrypt.org)
     staging API:

      ```yaml
      ---
      apiVersion: cert-manager.io/v1
      kind: ClusterIssuer
      metadata:
        name: letsencrypt-staging
      spec:
        acme:
          email: certificate-notices@my-company.com
          privateKeySecretRef:
            name: letsencrypt-production
          server: https://acme-staging-v02.api.letsencrypt.org/directory
          solvers:
            - http01:
                ingress:
                  class: contour
      ```

  2. Set `shared.ingress_issuer` to the name of your issuer

      ```yaml
      #! my-tap-values.yaml
      #! ...
      shared:
        ingress_issuer: letsencrypt-staging
      #! ...
      ```

  3. Apply Tanzu Application Platform installation values

      After the configuration is applied, components obtain certificates
      from the new issuer and serve them.

Other
: Complete the following steps

  You can use any other cert-manager-supported `ClusterIssuer` as an ingress
  issuer for Tanzu Application Platform.

  cert-manager supports a host of in-tree and out-of-tree issuers. See
  cert-manager's [documentation of
  issuers](https://cert-manager.io/docs/configuration/).

  1. Set `shared.ingress_issuer` to the name of your issuer

      ```yaml
      #! my-tap-values.yaml
      #! ...
      shared:
        ingress_issuer: my-company-ca
      #! ...
      ```

  1. Apply Tanzu Application Platform's installation values

      After the configuration is applied, components obtain certificates
      from the new issuer and serve them.

<br>

There are many ways and tools to assert that new certificates are
issued and served. It is best to connect to one of the ingress endpoints and
inspect the certificate it serves.

The `openssl` command-line utility is available on most
operating systems. The following command retrieves the certificate from an
ingress endpoint and shows its text representation:

```shell
# replace tap.example.com with your Tanzu Application Platform installation's ingress domain
openssl s_client -showcerts -servername tap-gui.tap.example.com -connect tap-gui.tap.example.com:443 <<< Q | openssl x509 -text -noout
```

Alternatively, use a browser to navigate to the ingress endpoint and click the
lock icon in the navigation bar to inspect the certificate.

## <a id="deactivate"></a>Deactivating TLS for ingress

While VMware does not recommend it, you can deactivate the ingress issuer by setting
`shared.ingress_issuer: ""`.

## <a id="override"></a>Overriding TLS for components

You can override TLS settings for each component. In your Tanzu Application Platform values file a component's configuration takes precedence over `shared` values. For more information about which components have ingress and how to configure them, see [components](../../../components.hbs.md)

>**Note** The approaches can be mixed. Use a shared ingress issuer, but
>override TLS configuration for select components.
