# Proposal G

Treat signatures special, add new pagination and reference annotations

## Description

This is inspired by ["Proposal F"](https://github.com/opencontainers/wg-reference-types/issues/50#issuecomment-1139100231), which suggests attaching references via the root index.

Signature data specifically should be stored as annotations to support the maximum number of clients, and should follow a convention pattern.

This proposal also extends the capabilities of the index to enable pagination and support
reference linking using new OCI-defined annotations (3 total) for more advanced use cases.

This proposal does not require any changes to the registry HTTP API.

## Links

| Description              | Link                                                           |
| ----------------------- | --------------------------------------------------------------- |
| Origin of "Proposal F"  | [View](https://github.com/opencontainers/wg-reference-types/issues/50#issuecomment-1139100231) |

## Modifications

### JSON Schema

The first page contains list of signatures and other sorts of things
you want supported by maximum # of clients.

As suggested in Proposal F, signature data is to be stored as annotations following some convention:

```json
{
    "schemaVersion": 2,
    "mediaType": "application/vnd.oci.image.index.v1+json",
    "manifests": [
        {
            "mediaType": "application/vnd.oci.image.index.v1+json", // Unsigned index
            "size": 7143,
            "digest": "sha256:e692418e4cbaf90ca69d05a66403747baa33ee08806650b51fab815ad7fc331f"
        },
        {
            "mediaType": "application/vnd.oci.image.manifest.v1+json", // Signature 1
            "size": 7143,
            "digest": "sha256:e692418e4cbaf90ca69d05a66403747baa33ee08806650b51fab815ad7fc331f",
            "annotations": {
                "dev.cosignproject.cosign/signature": "...",
                "dev.sigstore.cosign/bundle": "...",
                "dev.sigstore.cosign/certificate": "...",
                "dev.sigstore.cosign/chain": "...",
            }
        },
        {
            "mediaType": "application/vnd.oci.image.manifest.v1+json", // Signature 2
            "size": 7143,
            "digest": "sha256:e692418e4cbaf90ca69d05a66403747baa33ee08806650b51fab815ad7fc331f",
            "annotations": {
                "dev.pistachioproject.conesign/signature": "...",
                "dev.rainbowconebut4computers.conesign/bundle": "...",
                "dev.rainbowconebut4computers.conesign/certificate": "...",
                "dev.rainbowconebut4computers.conesign/chain": "...",
            }
        },
        // ... more signatures
  ],
  "annotations": {
      "org.opencontainers.image.nextpage": "sha256:a6124841..."
  }
}
```

The following new OCI annotation on the index above is defined by this proposal:

- `org.opencontainers.image.nextpage`: digest of next page of this index

As sugegested in [this comment](https://github.com/opencontainers/wg-reference-types/issues/50#issuecomment-1143664805), the digests above could technically be duplicated, but there are also other ways this could be done (adding index annotations to the first item in the list, etc).

The next page (and any subsequent ones) might be reserved for larger artifacts:

```json
{
    "schemaVersion": 2,
    "mediaType": "application/vnd.oci.image.index.v1+json",
    "manifests": [
        {
            "mediaType": "application/vnd.oci.image.manifest.v1+json", // arm64 SBOM
            "digest": "sha256:5b0044a1244...",
            "size": 1024,
            "annotations": {
                "org.opencontainers.reference.type": "sbom",
                "org.opencontainers.reference.digest": "sha256:c3c5822....."
            }
        },
        {
            "mediaType": "application/vnd.oci.image.manifest.v1+json", // amd64 SBOM
            "digest": "sha256:5b0044a44d...",
            "size": 1024,
            "annotations": {
                "org.opencontainers.reference.type": "sbom",
                "org.opencontainers.reference.digest": "sha256:4ff3ca912....."
            }
        },
        // ... more references

  ]
}
```

The lack of `org.opencontainers.image.nextpage` annotation above
indicates there are no additional pages.

To support older clients, this second page can be also be tagged
something clever such as `<repo>:sha256-<digest>.page2`.

In addition, the following new OCI annotationss found on the manifest above are
proposed to be used by advanced clients for additional filtering and reference types support.
(based on ones defined in [Proposal D](https://github.com/opencontainers/wg-reference-types/blob/main/docs/proposals/PROPOSAL_D.md#json-schema)):

- `org.opencontainers.reference.type`: type of artifact
- `org.opencontainers.reference.digest`: digest of another manifest this artifact references

Note: The size of each page is only limited by that of the [4 megabyte limit](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#pushing-manifests) on uploaded manifests.

### Registry HTTP API

N/A
