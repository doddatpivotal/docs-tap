# Migration From Supply Chain Security Tools - Sign

This topic explains how you can migrate from Supply Chain Security Tools - Sign
to Supply Chain Security Tools - Policy.

In Tanzu Application Platform v1.4, the Image Policy Webhook is removed. If the
Image Policy Webhook was used with the previous Tanzu Application Platform
versions in your cluster, you must migrate the `ClusterImagePolicy` resource
from Image Policy Webhook to Policy Controller. For information about additional
features introduced in SCST - Policy, see [Configuring Supply Chain Security
Tools - Policy](configuring.md).

## <a id="enable-controller"></a> Enable Policy Controller on Namespaces

Policy Controller works with an opt-in system. Operators must update namespaces
with the label `policy.sigstore.dev/include: "true"` to the namespace resource
to enable Policy Controller verification.

```console
kubectl label namespace my-secure-namespace policy.sigstore.dev/include=true
```

>**Caution** Without a Policy Controller ClusterImagePolicy applied, there are
fallback behaviors where images are validated against the public Sigstore
Rekor and Fulcio servers by using a keyless authority flow. Therefore, if the
deploying image is signed publicly by a third-party using the keyless
authority flow, the image are admitted as it can validate against the public
Rekor and Fulcio. To avoid this behavior, develop, and apply a ClusterImagePolicy
that applies to the images being deployed in the namespace.

## <a id="cluster-image"></a> Policy Controller ClusterImagePolicy

The Policy Controller `ClusterImagePolicy` does not have a name.
Image Policy Controller required that the `ClusterImagePolicy` be named
`image-policy` and that there be only one `ClusterImagePolicy`. Multiple
Policy Controller `ClusterImagePolicies` are applied. During validation, all
`ClusterImagePolicy` that have an image `glob` pattern that matches the
deploying image is evaluated. All matched `ClusterImagePolicies` must be
valid. For a `ClusterImagePolicy` to be valid, at least one authority in the
policy must  validate the signature of the deploying image.

## <a id="exclude-ns"></a> Excluding Namespaces

The namespaces listed in `spec.verification.exclude.resources.namespaces[]`
must have `policy.sigstore.dev/include` set to `false` or not be set.
Therefore, they are exempted from Policy Controller validation.

**Image Policy Webhook:**

```yaml
---
apiVersion: signing.apps.tanzu.vmware.com/v1beta1
kind: ClusterImagePolicy
metadata:
  name: image-policy
spec:
  verification:
    ...

    exclude:
      resources:
        namespaces:
        - image-policy-system
        - kube-system
        - cert-manager

    ...
```

## <a id="public-key"></a> Specifying Public Keys

`spec.verification.keys[].publicKey` from Image Policy Webhook is mapped to
`spec.authorities[].key.data` for Policy Controller.

The `name` associated with each `key` is no longer required. Image Policy Webhook
has direct association between `key` name and `imagePattern`. For Policy
Controller, multiple `ClusterImagePolicy` resources are defined to create
direct association between image patterns and key authorities.

Image patterns and keys are scoped to each `ClusterImagePolicy` resource.

Therefore, to have direct association be isolated between `key` and
`imagePattern`, multiple Policy Controller `ClusterImagePolicy` must be created.
Each `ClusterImagePolicy` has the image glob pattern defined and the
associated key authorities defined.

**Image Policy Webhook:**

```yaml
---
apiVersion: signing.apps.tanzu.vmware.com/v1beta1
kind: ClusterImagePolicy
metadata:
  name: image-policy
spec:
  verification:
    ...

    keys:
    - name: official-cosign-key
      publicKey: |
        -----BEGIN PUBLIC KEY-----
        MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEhyQCx0E9wQWSFI9ULGwy3BuRklnt
        IqozONbbdbqz11hlRJy9c7SG+hdcFl9jE9uE/dwtuwU2MqU9T/cN0YkWww==
        -----END PUBLIC KEY-----

    ...
```

**Policy Controller:**

```yaml
---
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: POLICY-NAME
spec:
  authorities:
  ...

  - key:
      data: |
        -----BEGIN PUBLIC KEY-----
        MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEhyQCx0E9wQWSFI9ULGwy3BuRklnt
        IqozONbbdbqz11hlRJy9c7SG+hdcFl9jE9uE/dwtuwU2MqU9T/cN0YkWww==
        -----END PUBLIC KEY-----

  ...
```

Where `POLICY-NAME` is the name of the cluster image policy you want to use.

## <a id="img-matching"></a> Specifying Image Matching

`spec.verification.images[].namePattern` from Image Policy Webhook maps to
`spec.images[].glob` for Policy Controller.

Policy Controller follows more closely to `glob` matching. For the Image Policy
Webhook, `registry.com/*` wildcards all projects and images under the
registry. However, `glob` matching uses `/` separator delimiting. Therefore,
the `glob` wildcard matching equivalent is `registry.com/**/*`. The `**` allows
for recursive project path matching while the trailing `*` images found in the
terminating project path.

If only one level of pathing is required, the `glob` pattern is
`registry.com/*/*`.

Policy Controller has defaults defined. If `*` is specified, the `glob`
matching behavior is `index.docker.io/library/*`. If `*/*` is specified,
the `glob` matching behavior is `index.docker.io/*/*`. With these defaults,
the `glob` pattern `**` matches against all images.

**Image Policy Webhook:**

```yaml
---
apiVersion: signing.apps.tanzu.vmware.com/v1beta1
kind: ClusterImagePolicy
metadata:
  name: image-policy
spec:
  verification:
    ...

    images:
    - namePattern: gcr.io/projectsigstore/cosign*
      keys:
      - name: official-cosign-key
      secretRef:
        name: your-secret
        namespace: your-namespace

    ...
```

**Policy Controller:**

```yaml
---
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: POLICY-NAME
spec:
  images:
  - glob: gcr.io/projectsigstore/cosign*
```

Where `POLICY-NAME` is the name of the cluster image policy you want to use.

## <a id="img-matching"></a> Specifying policy mode

If `AllowUnmatchedImages` is set to `true` in the Image Policy Webhook deployment,
create the following policy in the cluster:

```yaml
---
apiVersion: policy.sigstore.dev/v1beta1
kind: ClusterImagePolicy
metadata:
  name: allow-unmatched-image-policy
spec:
  images:
  - glob: "**"
    authorities:
    - static:
        action: pass
```
