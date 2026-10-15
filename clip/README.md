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

## Public installation

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
a37b8c95e6bd4fcfcaa072808f62af0772b2704f031e0d6c62eae31fdc88bee9  install-clip
SUMS
bash install-clip clip-1.0.0 /usr/local/bin
printf 'Staging retained: %s\n' "$stage"
)
```

Verify: clip --version; sha256sum /usr/local/bin/clip; and in an OSC 52
terminal: printf 'clipboard test\n' | clip. Check type -a clip before install.
The installer refuses symlinks and existing destinations, enforces trusted
staging permissions and checksum, and never executes the payload itself.

## GitHub Release

Current: [clip-v1.0.0-r2](https://github.com/tukdex/tuk-binaries/releases/tag/clip-v1.0.0-r2)
with four assets: clip-1.0.0; clip-1.0.0.sha256 (copy of 1.0.0.sha256);
clip-1.0.0-LICENSE.md (copy of LICENSE.md); and install-clip. The clip script is
byte-identical to 1.0.0 (SHA-256 above); r2 only corrects the installer, which now
accepts exactly one adjacent manifest named either 1.0.0.sha256 or
clip-1.0.0.sha256 and refuses if both exist. Release installs need no renaming.

[clip-v1.0.0](https://github.com/tukdex/tuk-binaries/releases/tag/clip-v1.0.0) is
superseded: its installer only accepts 1.0.0.sha256, so its manifest asset needed
renaming. Its assets are left unchanged; use r2. Release assets are never
replaced in place; a correction gets a new revision tag.

## clip 1.1.0 (prepared, not yet released)

Adds `clip run` for capturing command output including stderr. Piped mode is
unchanged. The 1.0.0 files and release above are unchanged; the Tuk catalogue
pins 1.0.0 until 1.1.0 is reviewed and published. After release, verify these
files from `clip/` (the installer is `install-clip-1.1.0` in Git, `install-clip`
as a release asset) against checksums you obtained independently:

    863e2d21b15896da5d2804dce34be78a270436f612a380462ab9df3e650774d6  clip-1.1.0
    13a5294330817e69d8dadbf414f9d71462211ba25168cf14a2c5ff5c4b1f0142  1.1.0.sha256
    e8f14a93d830587fca1511c5bf798fe291f5f2783b3b8ca46c2cbd7b9aa9581c  LICENSE.md
    dabcf60a9333b2f93cab866547bfd8e4b6db3ea3c4d7fa01dcaa71e6405d4522  install-clip-1.1.0

Install exactly as for 1.0.0 (staging, verification, no overwrite), with
`bash install-clip-1.1.0 clip-1.1.0 /usr/local/bin`. Requires GNU `tee`,
`mktemp` and `stat` (GNU coreutils) as well as `base64`.

### clip 1.1.0 usage

    command | clip                 # print stdout and copy it (stderr is NOT captured)
    command |& clip                # Bash shortcut for 2>&1: copy stdout and stderr
    command | clip --quiet         # copy only
    clip run -- COMMAND [ARGS...]  # run COMMAND, show and copy stdout+stderr
    clip run --quiet -- COMMAND [ARGS...]
    clip --version                 # clip 1.1.0
    clip --help

Examples:

    clip run -- ls -la /tmp
    clip run -- git status
    clip run -- bash -c 'echo normal; echo error >&2; exit 42'   # clip exits 42

`command | clip` only sees stdout: when a command fails and writes its
diagnostic to stderr, clip receives nothing to copy. In Bash, `|&` already
fixes that. `clip run` does the same and also returns the command's own exit
status, without depending on `set -o pipefail`.

#### How `clip run` behaves

- **Direct execution.** COMMAND and its arguments are passed as an argument
  vector to `exec`. No shell parses them, so spaces, quotes and metacharacters
  are ordinary bytes. Because the command is `exec`ed, shell functions and
  builtins are never used (`clip run -- echo` runs the external `echo`). Name a
  shell explicitly (`bash -c '...'`) if you want one. The `--` is required.
- **One merged stream.** stderr is redirected to stdout (`2>&1`) before capture,
  so the display and the copy contain the writes in the order they reached the
  merged stream. Many programs buffer stdout when it is not a terminal, so the
  observable order can differ from what you see when they run on a terminal.
- **Bytes preserved.** Output goes through `tee` into a private (mode 0600)
  temporary file created by `mktemp` in `$TMPDIR` (default `/tmp`), never into
  a shell variable, so NUL bytes, binary data, Unicode and trailing newlines
  survive. The file is deleted on normal exit and after SIGINT, SIGTERM or
  SIGHUP. Output is not logged anywhere else.
- **Live display.** Output appears while the command runs. `--quiet` suppresses
  this echo only; Clip's own diagnostics on stderr are never suppressed.
- **stdin** is not touched and is inherited by the command. The command's
  stdout and stderr are pipes, so programs that need a terminal for output
  (pagers, editors, `top`, `ssh -t`) may behave differently or not work; run
  those directly. A background process that keeps the output open delays
  completion.
- **Empty output** leaves the clipboard unchanged (an empty OSC 52 payload would
  clear it); Clip says so on stderr unless `--quiet` is given.
- **Size limit.** Output larger than `CLIP_MAX_BYTES` (default 75000 bytes, i.e.
  100000 Base64 characters) is displayed in full but not copied, and Clip says
  so. Raise the limit only if your terminal accepts larger OSC 52 payloads. The
  temporary file is capped just above the limit, so a runaway command cannot fill
  the disk through Clip. Piped mode (`| clip`) has no limit.
- **Error text.** If COMMAND cannot be executed, the shell's message
  (`...: exec: NAME: not found`, `Permission denied`) is part of the captured
  output and names Clip's script path.

#### Exit status of `clip run`

| Command | Clipboard | Exit status |
|---|---|---|
| exits 0 | delivered | 0 |
| exits N (nonzero) | delivered | N |
| exits 0 | failed or refused | 125 and a `clip:` diagnostic on stderr |
| exits N (nonzero) | failed or refused | N and a `clip:` diagnostic on stderr |

Statuses follow the shell: 127 command not found, 126 cannot execute (permission
denied, or a directory), 128+N when the command is killed by signal N. A command
that itself exits 125 cannot be told apart from a Clip failure by status alone;
read stderr. Usage errors and an invalid `CLIP_MAX_BYTES` exit 2 and run nothing.
Clip prints "clipboard was not set" whenever it did not write the OSC 52 sequence;
a written sequence still cannot prove the terminal accepted it.

Ctrl-C reaches both the command and Clip. Clip then still copies whatever the
command produced and returns its status (usually 130). A signal sent to Clip's
process alone is acted on only after the command finishes.
