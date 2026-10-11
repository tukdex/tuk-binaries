# tuk: bootstrap and utility manager

`tuk` installs a small set of reviewed utilities from pinned releases of this
repository. It is Bash plus a declarative catalogue; there is no daemon,
scheduler, remote service or package-manager database.

## Quick start (Debian, Proxmox and similar Linux)

```sh
git clone https://github.com/tukdex/tuk-binaries.git
cd tuk-binaries
./bootstrap
```

`./bootstrap` installs `tuk`, then (on a terminal) shows the tool list and asks
which to install. Nothing is installed unless you choose it. Afterwards:

```sh
tuk list                 # tools, versions and installation state
tuk install clip         # or: tuk install clip zellij
tuk status               # installed versions, hashes and any drift
tuk doctor               # read-only checks of prerequisites and catalogue
tuk install --dry-run clip   # show what would be fetched; changes nothing
tuk --version
tuk --help
```

Bootstrap validates the catalogue and your tool choices before it changes anything.

Scripted provisioning: `./bootstrap --install clip,zellij` (add `--no-prompt`
to guarantee no questions). Without `--install`, a non-interactive run installs
the manager only.

Requirements: Linux, Bash 4.4+, `curl`, GNU coreutils, `flock` (util-linux) and
either root or `sudo`. Bootstrap reports every missing prerequisite at once; on
Debian: `apt-get install curl coreutils util-linux sudo`.

## What it changes

| Path | Purpose |
|---|---|
| `/usr/local/bin/tuk` | the manager (mode 0755, root-owned) |
| `/usr/local/lib/tuk/packages.conf` | the catalogue copied from this checkout |
| `/var/lib/tuk/installed.tsv` | root-controlled installation record |
| `/usr/local/bin/<tool>` | each tool you install |

`./bootstrap` runs `tuk/tuk` from this checkout as root (through `sudo`) once, to
install it: review or pin the checkout first (see below). Nothing is downloaded by `./bootstrap` itself. `tuk install` makes HTTPS requests
only for GitHub release downloads (`github.com`, which redirects to GitHub's download hosts) of the tools you named.

## How installation stays safe

1. **Pinned, reviewed hashes.** The catalogue lists the exact release tag, asset
   names, sizes and SHA-256 values. The hashes live in this Git history, so the
   trust anchor is the commit you reviewed, not a checksum fetched next to a
   download. `tuk` never installs "whatever is Latest".
2. **Unprivileged fetch, privileged install.** Downloads go to a private
   directory and are checked as a first pass. Only the final step runs as root
   (via `sudo`): it copies the files into a root-only directory, **re-hashes the
   copies**, and only then runs the pinned installer.
3. **Reviewed installers are reused.** Each tool's existing `install-<name>`
   script (itself pinned by hash) performs the install. It refuses symlinks,
   untrusted ownership or permissions and existing destinations, and never runs
   the payload. `tuk` writes the checksum manifest it consumes from the
   catalogue, not from the download.
   `tuk install a b` fetches and verifies every requested tool before installing
   any, so a download or hash failure installs none. If the privileged step
   refuses a later tool, earlier ones stay installed and are listed in the error.
4. **No overwrites.** An existing file or symlink at the destination is never
   replaced, even if identical. A different installed version is never replaced
   automatically (upgrades are not implemented).
5. **Verified before reported.** `tuk` re-checks the installed bytes, mode (0755)
   and owner, and only then writes the record and says "Installed".
6. **Catalogue is data.** `packages.conf` is parsed line by line with strict
   per-field patterns. It is never sourced or evaluated; unknown keys, bad
   hashes or odd characters make the whole file invalid. Download URLs are built
   from a fixed base plus the validated tag and asset name.

## States

`tuk list` / `tuk status` report one of: `not-installed`; `verified`;
`modified` (content, mode or owner differs from the record, or the path became a
symlink); `missing` (recorded but gone); `outdated` (catalogue now lists a
different version); `unmanaged` (a command exists that tuk did not install; for
example one you installed by hand, even if byte-identical); `unsupported`
(OS/architecture) and `missing-dependency`. `tuk status` exits 1 when anything is
`modified` or `missing`.

## Pinning the bootstrap

`main` is a mutable branch. For servers you care about, install from a commit you
have reviewed instead of whatever `main` is today:

```sh
git clone https://github.com/tukdex/tuk-binaries.git
cd tuk-binaries
git checkout --detach <reviewed-commit-sha>
git rev-parse HEAD        # must print the reviewed value, obtained independently
sha256sum bootstrap tuk/tuk tuk/packages.conf   # optional: compare with your review notes
./bootstrap
```

Read `bootstrap`, `tuk/tuk` and `tuk/packages.conf` first; they are short. Because
`./bootstrap` copies the catalogue from the checkout, a pinned commit also pins
every tool hash.

## Adding a tool (maintainers)

Add a `[name]` section to `packages.conf` with the reviewed release's values (see
the existing entries). The installer asset must be a reviewed `install-<name>`
that accepts `SOURCE DEST_DIR`, reads an adjacent manifest (named by `manifest`,
containing `<sha256>  <payload name>`), refuses to overwrite, never executes the
payload, and installs `DEST_DIR/<name>` with mode 0755. `<name>` is the command
name. Do not list tools that depend on a private environment. Changes to the
catalogue are changes to what servers will run: review them like code.

## Upgrades and rollback (not yet implemented)

The record already stores version, hash, path and release per tool. A future
`tuk update NAME` would require the installed file to match its record, move it
to root-only storage, install the new pinned version through the same pipeline,
and keep the old file so `tuk rollback NAME` can restore it after re-checking its
hash. Until then, upgrade by hand: move the old file aside, then `tuk install`.

## Manager upgrades and removal

Re-running `./bootstrap` is safe: identical files are left alone, and a different
`tuk` is refused. `./bootstrap --upgrade-manager` replaces an older tuk only if
it is exactly what tuk previously recorded (never a locally edited or foreign
file). Remove a tool with `rm /usr/local/bin/<tool>`, then delete its line from
`/var/lib/tuk/installed.tsv` (or leave it; the state will read `missing`).

## Limitations

SHA-256 pins prove the bytes equal the reviewed build, not that it is defect-free.
`tuk` assumes the destination directory is root-controlled. Tool licences are in
their directories in this repository; `tuk` does not install them. The `sudo`
path is verified by review, not by the automated tests, which run unprivileged
against a scratch prefix.
