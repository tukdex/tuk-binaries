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

Requirements: Linux, Bash 4.4+, `curl`, GNU coreutils (including `timeout`), `flock`
(util-linux) and either root or `sudo` (`sudo` authenticates once, up front). Bootstrap reports every missing prerequisite at once; on
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
   any, so a download or hash failure installs none. Installation itself is then
   sequential and **not atomic**: if the privileged step refuses a later tool,
   earlier tools stay installed and recorded, and the error lists them (`installed
   before the failure: ...`). Scripted provisioning should check the exit status
   and re-run `tuk install` for the remainder.
4. **No overwrites.** An existing file or symlink at the destination is never
   replaced, even if identical. A different installed version is never replaced
   automatically (upgrades are not implemented).
5. **Verified before reported.** `tuk` re-checks the installed bytes, mode (0755)
   and owner, and only then writes the record and says "Installed". If that check
   fails, nothing is recorded. The new file is moved aside (quarantined, made
   non-executable, named `.<tool>.failed.*` in the same directory) only when it can
   be attributed to this attempt: the installer succeeded for a destination that
   did not exist, and the file is a regular file owned by the installing user, has
   a single link and was changed after the attempt began. Otherwise it is left in
   place and the failure is reported prominently; tuk then treats it as
   `unmanaged`. tuk never deletes it.
6. **Trusted locations.** Before reading the catalogue or record, or installing into
   `/usr/local/bin`, `tuk` checks every directory from `/` down: each must be
   owned by root and not writable by group or others (a root-owned sticky
   directory such as `/tmp` is tolerated as an ancestor). Symlinks on the path
   must be root-owned and sit in trusted directories, and the resolved path is
   checked as well. Anything else is refused with the offending path named and
   exact commands to inspect it. This happens before any download, and
   `./bootstrap` checks the destinations before it asks for `sudo`. tuk never moves or changes permissions
   or ownership of system directories itself.
7. **Catalogue is data.** `packages.conf` is parsed line by line with strict
   per-field patterns. It is never sourced or evaluated; unknown keys, bad
   hashes or odd characters make the whole file invalid. Download URLs are built
   from a fixed base plus the validated tag and asset name.

## States

`tuk list` / `tuk status` report one of: `not-installed`; `verified`;
`modified` (content, mode or owner differs from the record, or the path became a
symlink); `missing` (recorded but gone); `outdated` (catalogue now lists a
different version); `unmanaged` (a command exists that tuk did not install; for
example one you installed by hand, even if byte-identical); `unsupported`
(OS/architecture) and `missing-dependency`.

`tuk status` exit status: **0** when no tool is `modified` or `missing` and the
manager is `verified` (or is not installed because you are running from a
checkout); **1** when a tool or the manager is `modified` or `missing`, or the
installed manager has no installation record (its integrity cannot be verified).
`unmanaged`, `outdated` and `unsupported` tools are informational and do not
change the exit status.

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

Re-running `./bootstrap` is safe: identical files are left alone. If the installed
`tuk` or catalogue differs from this checkout, bootstrap stops and says which one
changed; replacing the catalogue changes which releases and SHA-256 values the
host trusts, so that always needs the explicit `./bootstrap --upgrade-manager`
after you have reviewed the checkout. The flag replaces a file only if it is exactly
what tuk previously recorded (right content, mode 0755/0644, root-owned); an edited,
foreign or unrecorded file is never replaced, with or without the flag. Both files
are decided before either is touched. If the manager step then fails, a catalogue
this run created is removed again and a catalogue this run replaced is restored.
This is best-effort cleanup, **not an atomic transaction**: a failure after the
manager file itself has been swapped is not rolled back; re-run `./bootstrap
--upgrade-manager` once the cause is fixed.

There is **no uninstall command** yet. If you delete a tool by hand (`rm
/usr/local/bin/<tool>`), `tuk` keeps its installation record, reports the tool as
`missing` and `tuk status` exits 1 until you run `tuk install <tool>` again.
Editing the root-controlled record by hand is not supported.

## Limitations

SHA-256 pins prove the bytes equal the reviewed build, not that it is defect-free.
Directories that are not root-controlled are refused rather than worked around.
Some systems ship `/usr/local` directories as `root:staff` mode 2775
(group-writable). `tuk` and the tool installers stop, before downloading anything,
and print the directory, its group and the exact commands to inspect and (if you
accept it) tighten it. `tuk` never changes system directory permissions itself.

Tool licences are in their directories in this repository; `tuk` does not install them. The `sudo`
path is verified by review, not by the automated tests, which run unprivileged
against a scratch prefix.
