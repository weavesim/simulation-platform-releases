# weavesim platform: releases

Release notes, the launcher, and the update manifest for the platform. This repository holds **no source code**. The only image published is the small installer, `ghcr.io/weavesim/platform-installer`; the platform's own image is built on your machine from the release's files.

> **Status:** the first release, [0.3.0](https://github.com/weavesim/simulation-platform-releases/releases/tag/v0.3.0), is out. It is an early release: only Linux with Docker has been tried. Read [docs/install.md](docs/install.md) to install.

## Install

You need Docker or Podman. Then, in a terminal:

```
mkdir -p ~/bin
curl -fsSL https://raw.githubusercontent.com/weavesim/simulation-platform-releases/main/launcher/platform -o ~/bin/platform
chmod +x ~/bin/platform
~/bin/platform update
~/bin/platform up
```

With **Podman**, start its socket once first: `systemctl --user enable --now podman.socket` (details for Docker and Podman: [docs/install.md](docs/install.md#docker-or-podman)). The first `update` builds the platform on your machine and takes a few minutes. `up` prints the address to open in your browser. The full guide, with updating, mirrors and checking a release by hand: [docs/install.md](docs/install.md). You do not need to download any of the release files by hand.

## What is here

- `CHANGELOG.md`: the release notes.
- `LICENSE`: the end-user licence. The platform's programs are not open source; they are free to use under it.
- `launcher/`: the `platform` script that starts, stops and updates the platform (Linux and macOS; Windows has no installer commands yet).
- `cosign.pub`: the public keys that sign each release's manifest and `SHA256SUMS`.
- `docs/`: install (with update, mirrors and verifying a release) and migrate guides.
- GitHub Releases: the release's files, `latest.json` and `SHA256SUMS` with their signatures.

## Reporting a problem

- **Bugs:** open an issue and fill in the form. Include the output of `platform --version`. **Do not attach confidential models, project files, keys or tokens.**
- **Security problems:** use the private reporting link on the Security tab. Do not open a public issue.
- **Anything that cannot be public:** write to **weavesim@umitas.com**.
