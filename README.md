# DATA Compass — Release Artifact Mirror

This is the public mirror for DATA Compass release artifacts. The source repository
(`compassfordata/data-compass`) is private. Compiled release bundles are mirrored
here so customers can download them without GitHub authentication.

## License

DATA Compass is commercial software distributed under the DATA Compass End User
License Agreement (EULA). The EULA is presented during installation and bundled
with each release. Source code is private; binary artifacts in this repository
are licensed only for use under the EULA terms. For commercial licensing
inquiries: <https://compassfordata.com> or `support@compassfordata.com`.

## Verifying release artifacts

Every `.tar.gz`, the `compass-setup.sh` installer, and every SBOM in this repo
is signed with **cosign keyless** via Sigstore (Fulcio for issuance, Rekor for
the transparency log). Each signed artifact ships with two sidecar files:

- `<artifact>.sig` — the cosign signature
- `<artifact>.cert` — the X.509 certificate (encodes the signer identity)

To verify a release artifact:

```bash
cosign verify-blob \
  --certificate compass-deploy-vX.Y.Z.tar.gz.cert \
  --signature   compass-deploy-vX.Y.Z.tar.gz.sig \
  --certificate-identity-regexp 'https://github\.com/compassfordata/compass-releases/\.github/workflows/(backfill|publish)\.yml@refs/heads/main' \
  --certificate-oidc-issuer     'https://token.actions.githubusercontent.com' \
  compass-deploy-vX.Y.Z.tar.gz
```

A successful verification prints `Verified OK`. Any other output means
verification failed — do not trust the artifact.

The `--certificate-identity-regexp` above asserts that the signature was
produced by one of two workflows in this repository, running on `main`. The
`--certificate-oidc-issuer` asserts that GitHub Actions' OIDC provider issued
the certificate.

After signature verification, also verify the artifact's checksum:

```bash
sha256sum -c compass-deploy-vX.Y.Z.tar.gz.sha256
```

The SBOM (`*.cdx.json`) is generated from the **unpacked tarball contents**
(not from the source tree at release time), and is itself cosign-signed so
its contents cannot be silently swapped. Verify it the same way:

```bash
cosign verify-blob \
  --certificate compass-deploy-vX.Y.Z.cdx.json.cert \
  --signature   compass-deploy-vX.Y.Z.cdx.json.sig \
  --certificate-identity-regexp 'https://github\.com/compassfordata/compass-releases/\.github/workflows/(backfill|publish)\.yml@refs/heads/main' \
  --certificate-oidc-issuer     'https://token.actions.githubusercontent.com' \
  compass-deploy-vX.Y.Z.cdx.json
```

## Supported versions

Only the latest minor release receives security patches. Older releases remain
available for compatibility with existing deployments but are not maintained.
Customers are expected to upgrade within **90 days** of a new minor release.

| Version | Status | Notes |
|---|---|---|
| `v1.1.0` | **Supported** | Current production release |
| `v1.0.0` | **Deprecated** | Contains known high-severity vulnerabilities patched in v1.1.0+. Available for compatibility only. Upgrade as soon as possible. |

## Security disclosures

For security issues affecting any released artifact, email
`security@compassfordata.com`. Do not file public issues for vulnerabilities.
We follow coordinated disclosure.

## Reporting issues

- **Source code or product issues:** file in `compassfordata/data-compass` (you
  may need to be granted access — contact `support@compassfordata.com`).
- **Mirror publishing issues** (signature failures, missing artifacts, broken
  SBOMs): file here.

## What is and isn't here

| Artifact | Location |
|---|---|
| Deploy bundle (`compass-deploy-v*.tar.gz`) | This repo, per release |
| Offline (air-gapped) bundle (`compass-offline-v*.tar.gz`) | This repo, per release where applicable |
| Online installer script (`compass-setup.sh`) | This repo, per release |
| Cosign signatures (`*.sig`) and certs (`*.cert`) | This repo, per release |
| CycloneDX SBOMs (`*.cdx.json`) | This repo, per release |
| Checksums (`*.sha256`) | This repo, per release |
| Docker image | `ghcr.io/compassfordata/data-compass:<tag>` (separate, also cosign-signed) |
| Source code | `compassfordata/data-compass` (private) |
