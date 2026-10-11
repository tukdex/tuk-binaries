# tuk-binaries

## Quick start

```sh
git clone https://github.com/tukdex/tuk-binaries.git
cd tuk-binaries
./bootstrap          # installs the tuk manager; nothing else unless you choose
tuk list
tuk install clip zellij
```

[tuk](tuk/README.md) installs only the pinned, reviewed releases listed in its
catalogue, verifies SHA-256 values and never overwrites existing commands. `main`
is mutable: to install from a reviewed commit, see
[pinning the bootstrap](tuk/README.md#pinning-the-bootstrap).

Precompiled Linux executables (Zellij) and a standalone Bash script (clip).
Zellij is a **locally compiled redistribution, not an official upstream
artefact**; its executables are release assets only, not stored in Git.
clip is versioned, auditable Bash source in Git, with its own checksum and installer.

| Tool | Release | Platform |
|---|---|---|
| [Zellij](zellij/README.md) 0.45.1 | `zellij-binary-v0.45.1-r2` | Linux x86-64, static PIE |
| [clip](clip/README.md) 1.0.0 | [`clip-v1.0.0-r2`](https://github.com/tukdex/tuk-binaries/releases/tag/clip-v1.0.0-r2) | Linux, Bash, GNU base64 |

Always verify a download against a checksum you obtained independently, and never
pipe a download into a shell. The installers refuse to overwrite existing files.
No warranty is provided; see each tool's licence.
