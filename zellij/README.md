# Zellij 0.45.1 (Linux x86-64)

Locally compiled static PIE build of [Zellij](https://zellij.dev) 0.45.1, MIT
licensed (see [LICENSE.md](LICENSE.md)). Not an official Zellij release artefact.

Release assets (`https://github.com/tukdex/tuk-binaries/releases/tag/zellij-binary-v0.45.1`):

- `zellij-0.45.1-linux-x86_64-static`
- `zellij-0.45.1.sha256`: `d006c521dcb475a6005d741e9dd7c5758e5a23b28dd60a5c10cebfa4876319dd`
- `zellij-0.45.1-LICENSE.md`
- `install-zellij`: checksum-verifying installer (no download, no overwrite)

Install, as root, from a new root-only 0700 directory (needs curl, Bash, GNU coreutils):

```sh
umask 077; cd "$(mktemp -d /root/zellij-stage.XXXXXX)"
u=https://github.com/tukdex/tuk-binaries/releases/download/zellij-binary-v0.45.1
for f in zellij-0.45.1-linux-x86_64-static zellij-0.45.1.sha256 zellij-0.45.1-LICENSE.md install-zellij; do
  curl --proto '=https' --tlsv1.2 -fSLo "$f" "$u/$f"; done
mv zellij-0.45.1.sha256 0.45.1.sha256
echo 'd006c521dcb475a6005d741e9dd7c5758e5a23b28dd60a5c10cebfa4876319dd  zellij-0.45.1-linux-x86_64-static' | sha256sum -c -
less install-zellij   # review before running
bash install-zellij zellij-0.45.1-linux-x86_64-static /usr/local/bin
zellij --version
```

An existing `zellij` is never overwritten; move it aside first. Remove with `rm /usr/local/bin/zellij`.
