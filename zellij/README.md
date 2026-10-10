# Zellij 0.45.1 (Linux x86-64)

Locally compiled static PIE build of [Zellij](https://zellij.dev) 0.45.1, MIT
licensed (see [LICENSE.md](LICENSE.md)). Not an official Zellij release artefact.

Release assets (`https://github.com/tukdex/tuk-binaries/releases/tag/zellij-binary-v0.45.1`):

- `zellij-0.45.1-linux-x86_64-static`
- `zellij-0.45.1.sha256`: `d006c521dcb475a6005d741e9dd7c5758e5a23b28dd60a5c10cebfa4876319dd`
- `zellij-0.45.1-LICENSE.md`
- `install-zellij`: checksum-verifying installer (no download, no overwrite)

Install as root (needs only curl, Bash, GNU coreutils; no GitHub login). Every
download is checked against the pinned SHA-256 values below **before** anything
runs, in a fresh root-only 0700 directory. Compare the pins with values you
obtained independently (for example from the maintainers' reviewed source) rather
than only trusting this page. The subshell stops at the first failure without
closing your SSH session.

```sh
(
set -euo pipefail; umask 077
[ "$(id -u)" = 0 ] || { echo 'run as root' >&2; exit 1; }
stage=$(mktemp -d /root/zellij-stage.XXXXXX); cd "$stage"
u=https://github.com/tukdex/tuk-binaries/releases/download/zellij-binary-v0.45.1
for f in zellij-0.45.1-linux-x86_64-static zellij-0.45.1.sha256 zellij-0.45.1-LICENSE.md install-zellij; do
  curl --proto '=https' --tlsv1.2 -fSLo "$f" "$u/$f"
done
# Verify everything against pinned values BEFORE running anything.
sha256sum -c - <<'SUMS'
d006c521dcb475a6005d741e9dd7c5758e5a23b28dd60a5c10cebfa4876319dd  zellij-0.45.1-linux-x86_64-static
871987b1cd7dfc4050f3478238d167af5931631cee8f74aad3894b5d78c6102f  install-zellij
606009ef86a4ad18d34cf2621ed4f72ea063e374fb5484e81574f71c81bdc2f9  zellij-0.45.1.sha256
9aa6c363b18a48eeecbffe13ff390c76ede2c90eb37ba17dfbbaffc42892d87d  zellij-0.45.1-LICENSE.md
SUMS
mv zellij-0.45.1.sha256 0.45.1.sha256; mv zellij-0.45.1-LICENSE.md LICENSE.md
bash install-zellij zellij-0.45.1-linux-x86_64-static /usr/local/bin
echo "staging kept for reference/rollback: $stage"
)
```

Check afterwards: `sha256sum /usr/local/bin/zellij; stat -c '%U:%G %a' /usr/local/bin/zellij; zellij --version`.

An existing `zellij` is never overwritten; move it aside first. Remove with `rm /usr/local/bin/zellij`.
