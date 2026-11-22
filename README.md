# keelbox releases

Signed release binaries of [keelbox](https://keelbox.dev), a command-line tool that sets up an Ubuntu machine for AI coding agents such as Claude Code and Codex. The source is at [keelbox/keelbox](https://github.com/keelbox/keelbox); this repository holds release assets only.

## Install

```sh
curl -fsSL https://get.keelbox.dev | sh
```

The installer picks the binary for your OS and architecture, checks the release key's signature on `SHA256SUMS` and the binary against it (and the Sigstore signature, when `cosign` is installed), then installs it to `~/.local/bin`. Pin a version with `sh -s -- --version v0.1.0`. Update later with `keelbox update`.

## Assets

Each release has:

| Asset | |
|---|---|
| `keelbox-linux-amd64`, `keelbox-linux-arm64`, `keelbox-darwin-amd64`, `keelbox-darwin-arm64` | static binaries |
| `keelbox-<os>-<arch>.spdx.json` | SPDX SBOM per binary |
| `install.sh` | the installer, as served at get.keelbox.dev |
| `SHA256SUMS` | checksums of every asset above |
| `SHA256SUMS.sig`, `SHA256SUMS.sshsig` | the release key's Ed25519 signature of `SHA256SUMS`, raw and in SSH form |
| `SHA256SUMS.sigstore.json` | keyless cosign signature of `SHA256SUMS` |
| `stable.json` | the channel index published to get.keelbox.dev for this release |

## Verify by hand

The release key signs a line naming the version, followed by `SHA256SUMS`, so one release's checksums cannot pass as another's. Its public key is in [`internal/release/keys`](https://github.com/keelbox/keelbox/tree/main/internal/release/keys) in the source repository.

```sh
v=v0.1.0
{ printf 'keelbox-release %s\n' "$v"; cat SHA256SUMS; } > signed
openssl pkeyutl -verify -pubin -inkey keelbox-release-1.pem -rawin -in signed -sigfile SHA256SUMS.sig
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity https://github.com/keelbox/keelbox/.github/workflows/release.yml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com SHA256SUMS
sha256sum --check --ignore-missing SHA256SUMS
```

The cosign signer is keelbox's release workflow, identified by GitHub's OIDC token and logged in Sigstore's public transparency log.

## Security

Report vulnerabilities to security@keelbox.dev. Licence: Apache-2.0.
