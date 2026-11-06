# keelbox releases

Signed release binaries of [keelbox](https://keelbox.dev): agent boxes in your own cloud. This repository holds release assets only.

## Install

```sh
curl -fsSL https://get.keelbox.dev | sh
```

The installer picks the binary for your OS and architecture, verifies it against `SHA256SUMS` (and the signature, when `cosign` is installed) and installs it to `~/.local/bin`. Pin a version with `sh -s -- --version v0.1.0`. Update later with `keelbox self-update`.

## Assets

Each release has:

| Asset | |
|---|---|
| `keelbox-linux-amd64`, `keelbox-linux-arm64`, `keelbox-darwin-amd64`, `keelbox-darwin-arm64` | static binaries |
| `keelbox-<os>-<arch>.spdx.json` | SPDX SBOM per binary |
| `SHA256SUMS` | checksums of every asset above |
| `SHA256SUMS.sigstore.json` | keyless cosign signature of `SHA256SUMS` |
| `stable.json` | the channel index published to get.keelbox.dev for this release |

## Verify by hand

```sh
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity https://github.com/keelbox/keelbox/.github/workflows/release.yml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com SHA256SUMS
sha256sum --check --ignore-missing SHA256SUMS
```

The signer is keelbox's release workflow, identified by GitHub's OIDC token and logged in Sigstore's public transparency log. Binaries are reproducible from the tagged commit.

## Security

Report vulnerabilities to security@keelbox.dev. Licence: Apache-2.0.
