# tuk-binaries

Precompiled Linux executables (Zellij) and a standalone Bash script (clip).
Zellij is a **locally compiled redistribution, not an official upstream
artefact**; its executables are release assets only, not stored in Git.
clip is versioned, auditable Bash source in Git, with its own checksum and installer.

| Tool | Release | Platform |
|---|---|---|
| [Zellij](zellij/README.md) 0.45.1 | `zellij-binary-v0.45.1-r2` | Linux x86-64, static PIE |
| [clip](clip/README.md) 1.0.0 | Public source package; optional Release pending | Linux, Bash, GNU base64 |

Always verify a download against a checksum you obtained independently, and never
pipe a download into a shell. The installers refuse to overwrite existing files.
No warranty is provided; see each tool's licence.
