# Prime CLI

The terminal client for the Prime platform. Run it bare for an interactive TUI, or
pass subcommands for a scriptable, non-interactive path suitable for CI.

> **Snapshot builds.** Releases tagged `-snapshot` are preview cuts, not stable
> releases. They point at the Prime **staging** gateway by default, so treat any data
> you create with them as disposable. Command surfaces and flags can change between
> snapshots without notice. Override the gateway with `prime configure set
> --gateway-url <URL>` to talk to a different environment.

```
prime                          # interactive TUI
prime capability list          # machine path, table output
prime capability list -o json  # machine path, JSON output
```

This repository distributes prebuilt binaries only. Each release ships the CLI, shell
completions, and a self-contained HTML user guide covering every command.

## Download

Grab the archive for your platform from the [latest release](../../releases/latest).

| Platform | Architecture | Archive |
|---|---|---|
| macOS | Apple Silicon (M1–M4) | `prime-<version>-darwin-arm64.tar.gz` |
| Linux | x86-64 | `prime-<version>-linux-amd64.tar.gz` |
| Linux | ARM64 | `prime-<version>-linux-arm64.tar.gz` |
| Windows | x86-64 | `prime-<version>-windows-amd64.zip` |
| Windows | ARM64 | `prime-<version>-windows-arm64.zip` |

Binaries are statically linked with no runtime dependencies — no Go toolchain, no libc
version to match.

## Install

### macOS / Linux

```bash
tar -xzf prime-<version>-<platform>.tar.gz
cd prime-<version>-<platform>
./install.sh
```

`install.sh` puts `prime` on your PATH and registers zsh/bash/fish completions. It picks
an install prefix automatically; override it when you want a different one:

```bash
PREFIX=/opt/homebrew ./install.sh      # Apple Silicon Homebrew
PREFIX="$HOME/.local" ./install.sh     # user-local, no sudo
```

Open a new shell afterwards so PATH and completions load.

#### macOS: first-run security prompt

These binaries are **not code-signed or notarized**. macOS quarantines them on first
run and shows "cannot be opened because the developer cannot be verified". Clear the
quarantine flag after install:

```bash
xattr -d com.apple.quarantine "$(command -v prime)"
```

Alternatively, approve it once under System Settings → Privacy & Security.

### Windows

```powershell
# Extract, then move the folder somewhere permanent:
#   %LOCALAPPDATA%\Programs\prime

setx PATH "%PATH%;%LOCALAPPDATA%\Programs\prime"
```

Open a new terminal and run `prime version`. For tab completion in PowerShell, load the
bundled script — append this line to your `$PROFILE` to make it permanent:

```powershell
. $env:LOCALAPPDATA\Programs\prime\completions\prime.ps1
```

## Verify your download

Every release includes `SHA256SUMS`.

```bash
# macOS / Linux
sha256sum -c SHA256SUMS --ignore-missing
```

```powershell
# Windows
Get-FileHash prime-<version>-windows-amd64.zip -Algorithm SHA256
```

## First run

Snapshot builds already default to the staging gateway, so you only need to
authenticate. Browser login:

```bash
prime login
```

Or set an API key directly:

```bash
prime configure set --api-key <API_KEY>
```

To target a different environment, pass the gateway too:

```bash
prime configure set --gateway-url <URL> --api-key <API_KEY>
```

Confirm what you're talking to, then check the backend is healthy:

```bash
prime whoami
prime doctor
```

Configuration lives in `~/.prime` (`%USERPROFILE%\.prime` on Windows). Use
`--profile <name>` to keep several environments side by side.

## What's in the CLI

Run `prime docs` to open the bundled HTML guide, or `prime <command> --help` for any
single command.

| Area | Commands |
|---|---|
| Identity & config | `login` `configure` `whoami` `apikey` `iam` `tenant` |
| Conversations | `chat` `run` `mrun` `approval` |
| Capabilities | `capability` `authoring` `publication` |
| Knowledge & memory | `knowledge` `memory` |
| Models & quotas | `model` `eval` |
| Operations | `doctor` `usage` `billing` `cron` `channel` `workspace` `sandbox` |

Global flags worth knowing:

- `-o json` / `-o xml` — machine-readable output instead of tables
- `--columns a,b,c` — pick table columns on list commands
- `-q` — suppress stdout, rely on exit code (CI)
- `-v` — log request method/path and request-id to stderr
- `--no-color` — strip ANSI colors

Destructive writes require `--yes`, which is mandatory when stdout is not a TTY.
Complex create/update bodies come from `--file`/`-f`, stdin (`-f -`), or `--edit`.

## Uninstall

```bash
./uninstall.sh              # remove binary + completions
./uninstall.sh --purge      # also remove ~/.prime
```

On Windows, delete the install folder and remove it from PATH.

## Support

Open an issue in this repository for install or packaging problems. Include:

- `prime version`
- your OS and architecture
- the output of `prime doctor -v`, with any tokens redacted
