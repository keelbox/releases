# keelbox releases

Signed release binaries of [keelbox](https://keelbox.dev), a command-line tool that sets up an Ubuntu machine for AI coding agents such as Claude Code and Codex. This repository holds release assets only. The docs are at [keelbox.dev/docs](https://keelbox.dev/docs).

Every release so far is a release candidate for v0.2. They were built from a private repository, so the commit `keelbox version` reports for them is not public. The source is to be published at github.com/keelbox/keelbox with v0.2.

## Install

```sh
curl -fsSL https://get.keelbox.dev | sh
```

The installer picks the binary for your OS and architecture and installs it to `~/.local/bin` without sudo. Before it installs anything it checks the release key's signature on `SHA256SUMS`, with OpenSSL 3 or with `ssh-keygen`, and requires the binary's checksum to match. If `cosign` is installed it also checks the Sigstore signature. It is a POSIX `sh` script written to be read first: [get.keelbox.dev](https://get.keelbox.dev).

Pin a version with `sh -s -- --version v0.2.0-rc.7`. Update later with `keelbox self-update`. Homebrew: `brew install keelbox/tap/keelbox`. More in [installing keelbox](https://keelbox.dev/docs/guides/installing).

## Assets

| Asset | Content |
|---|---|
| `keelbox-linux-amd64`, `keelbox-linux-arm64`, `keelbox-darwin-amd64`, `keelbox-darwin-arm64` | Static binaries, no archive. |
| `keelbox-<os>-<arch>.spdx.json` | SPDX SBOM per binary. |
| `install.sh` | The installer served at get.keelbox.dev, the same file. |
| `SHA256SUMS` | Checksums of every binary, SBOM and `install.sh`. |
| `SHA256SUMS.sig` | Ed25519 signature by the keelbox release key over the line `keelbox-release <version>` followed by `SHA256SUMS`, so one release's file cannot pass as another's. |
| `SHA256SUMS.sshsig` | The same key's signature in `ssh-keygen -Y` form, namespace `keelbox-release`, for machines whose openssl lacks Ed25519 (macOS). |
| `SHA256SUMS.sigstore.json` | Keyless cosign bundle for `SHA256SUMS`, signed by the release workflow's GitHub Actions identity. An extra check. |
| `stable.json` | The channel index for this release. |

`v0.1.0-rc.1` was withdrawn. It predates the release key and current clients do not accept it.

## Verify by hand

The release public key, `keelbox-release-1`:

```text
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAwYkL/I8qupRfhwhMX/M7vsOYj0HXG/ErgT10EeiJlrU=
-----END PUBLIC KEY-----
```

Save it as `keelbox-release.pem`, then, with OpenSSL 3:

```sh
v=v0.2.0-rc.7; b=https://github.com/keelbox/releases/releases/download/$v
curl -fsSLO $b/SHA256SUMS -O $b/SHA256SUMS.sig -O $b/keelbox-linux-amd64
{ printf 'keelbox-release %s\n' "$v"; cat SHA256SUMS; } > signed
openssl pkeyutl -verify -pubin -inkey keelbox-release.pem -rawin -in signed -sigfile SHA256SUMS.sig
sha256sum --check --ignore-missing SHA256SUMS
```

The first check prints `Signature Verified Successfully`, the second `keelbox-linux-amd64: OK`.

Without OpenSSL 3 (macOS), download `SHA256SUMS.sshsig` instead and use the key's OpenSSH form:

```sh
echo 'keelbox-release namespaces="keelbox-release" ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMGJC/yPKrqUX4cITF/zO77DmI9B1xvxK4E9dBHoiZa1' > allowed_signers
ssh-keygen -Y verify -f allowed_signers -I keelbox-release -n keelbox-release -s SHA256SUMS.sshsig < signed
shasum -a 256 --check --ignore-missing SHA256SUMS
```

The Sigstore signature, with [cosign](https://docs.sigstore.dev/):

```sh
curl -fsSLO $b/SHA256SUMS.sigstore.json
cosign verify-blob --bundle SHA256SUMS.sigstore.json \
  --certificate-identity https://github.com/keelbox/keelbox/.github/workflows/release.yml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com SHA256SUMS
```

How releases are built and what each check covers: [the release reference](https://keelbox.dev/docs/reference/release).

## Security

Report vulnerabilities to security@keelbox.dev; see the [security policy](https://keelbox.dev/docs/security-policy). Licence: Apache-2.0.
