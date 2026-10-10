# clip 1.0.0 (Linux)

Standalone MIT-licensed Bash script for terminal clipboard support through OSC 52.
This public package contains only the generic source, installer, checksum and
licence; no private infrastructure configuration or operational data.

## Usage

    command | clip            # print the original output and copy it
    command | clip --quiet    # copy only (the old default)
    clip --version

Input is reproduced byte-for-byte on stdout. Clipboard escape bytes go to
/dev/tty, not stdout. Without a controlling terminal stdout is still printed,
but the operation fails nonzero as clipboard copying is unavailable. OSC 52
acceptance cannot be confirmed; terminal and multiplexer settings or size
limits may prevent it. Requires Linux, Bash and GNU base64 (-w0 and --decode).
Clipboard content is never redacted: avoid copying sensitive output.

## Public installation (after merging this PR)

Review the SHA-256 pins independently. As root, download to a protected
temporary directory and verify ALL files before running any downloaded code.
An existing clip is never overwritten.

```sh
(
set -euo pipefail; umask 077
[ "$(id -u)" = 0 ] || { echo 'run as root' >&2; exit 1; }
stage=$(mktemp -d /root/clip-stage.XXXXXX); cd "$stage"
u=https://raw.githubusercontent.com/tukdex/tuk-binaries/main/clip
for f in clip-1.0.0 1.0.0.sha256 LICENSE.md install-clip; do
  curl --proto '=https' --tlsv1.2 -fSLo "$f" "$u/$f"
done
sha256sum -c - <<'SUMS'
217eceba1dcc86e4ec3a9be2c85b3aaca75056f00bec0c7183332d457dcae1db  clip-1.0.0
5c12abfbe6ba1ff9905e4185b36dd18e385923af65d5e5f3aa761e644638611d  1.0.0.sha256
e8f14a93d830587fca1511c5bf798fe291f5f2783b3b8ca46c2cbd7b9aa9581c  LICENSE.md
dd62606a5d1ead43553cb8d460bf1178550efa47bd1ff2165b9a37248ff79cc3  install-clip
SUMS
bash install-clip clip-1.0.0 /usr/local/bin
printf 'Staging retained: %s\n' "$stage"
)
```

Verify: clip --version; sha256sum /usr/local/bin/clip; and in an OSC 52
terminal: printf 'clipboard test\n' | clip. Check type -a clip before install.
The installer refuses symlinks and existing destinations, enforces trusted
staging permissions and checksum, and never executes the payload itself.

## Optional GitHub Release

A separate versioned release can be published with tag clip-v1.0.0 and four
assets: clip-1.0.0; clip-1.0.0.sha256 (copy of 1.0.0.sha256);
clip-1.0.0-LICENSE.md (copy of LICENSE.md); and install-clip.
This release is NOT claimed published. Verify anonymous downloads and every
asset checksum before announcing it; do not replace release assets in place.
