# tuk-binaries

Precompiled Linux executables, published as versioned GitHub Releases, so they
can be installed without compiling. These are **locally compiled
redistributions, not official upstream artefacts**. Each release carries the
upstream licence, a SHA-256 manifest and the installer. Executables are release
assets only; they are not stored in Git.

| Tool | Release | Platform |
|---|---|---|
| [Zellij](zellij/README.md) 0.45.1 | `zellij-binary-v0.45.1` | Linux x86-64, static PIE |

Always verify a download against a checksum you obtained independently, and never
pipe a download into a shell. The installers refuse to overwrite existing files.
No warranty is provided; see each tool's licence.
