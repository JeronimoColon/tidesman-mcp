<p align="center">
  <img src="assets/logo.svg" width="128" alt="Tidesman logo">
</p>

<h1 align="center">Tidesman</h1>

<p align="center">
  A free native MCP server for running, understanding, and debugging Linux containers with Apple's <code>container</code> tool.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS%2026-Apple%20Silicon-103E63?style=flat-square" alt="macOS 26 · Apple Silicon">
  <img src="https://img.shields.io/badge/MCP-server-187E9E?style=flat-square" alt="MCP server">
  <img src="https://img.shields.io/badge/license-EULA-475569?style=flat-square" alt="License: EULA">
</p>

<p align="center">
  <a href="https://tidesman.dev">Website</a> &middot;
  <a href="https://github.com/JeronimoColon/tidesman-mcp/releases">Downloads</a>
</p>

It lets an AI assistant (Claude Desktop, Claude Code, or any MCP-compatible client) list,
run, inspect, and clean up Linux containers on your Mac, safely and with a full audit trail.

> This repository is the public download and documentation home for Tidesman. The source is
> maintained privately; the binaries published here are signed with an Apple Developer ID
> certificate and notarized by Apple.

## Demo

<!-- A short screen recording is added here with the first release. -->
<p align="center"><em>Demo coming with the first release.</em></p>

## What it is

<p align="center">
  <img src="assets/how-it-works.svg" width="860"
       alt="How Tidesman works: an AI client speaks MCP over stdio to Tidesman, which talks directly to Apple's container apiserver over XPC to run Linux containers in lightweight VMs.">
</p>

MCP (Model Context Protocol) is an open standard that lets an AI assistant call external
tools: small functions that do real work, such as listing or starting containers. Tidesman is
the server side of that standard: it offers a set of container tools, and the AI is the client
that calls them.

Apple's `container` is Apple's tool for running Linux containers on macOS. Normally you drive
it by typing commands in a terminal. Tidesman instead talks straight to the background service
that `container` relies on (its "apiserver"), using Apple's own Swift client library. Going
directly to that service is faster and sturdier than wrapping the command line.

## Requirements

- An Apple Silicon Mac running macOS 26 (Apple's container runtime requires both).
- Apple's `container` installed and started: `container system start`.

## Install

Tidesman is a signed, notarized binary. Pick whichever channel suits you.

### Homebrew (recommended)

```
brew install JeronimoColon/tidesman/tidesman
```

This adds the tap and installs `tidesman` to `/opt/homebrew/bin`.

### Installer package (.pkg)

Download `tidesman-<version>.pkg` from the
[latest release](https://github.com/JeronimoColon/tidesman-mcp/releases/latest), double-click
it, and follow the prompts. It installs `tidesman` to `/usr/local/bin`. The package is signed,
notarized, and stapled, so it verifies even offline.

### Direct binary (zip)

Download `tidesman-<version>-macos-arm64.zip`, unzip it, and move `tidesman` onto your PATH.
Because the bare binary is not stapled, macOS checks it online the first time you run it.

### Claude Desktop one-click (.mcpb)

Download `tidesman-<version>.mcpb` and open it with Claude Desktop (Settings → Extensions). The
bundle embeds the signed binary and registers the server for you in read-only mode.

### Verify your download

Every release ships a `SHA256SUMS` file. From the folder holding your downloads:

```
shasum -c SHA256SUMS
```

## Configure your MCP client

Tidesman runs as a local command your client launches over standard input/output. Point the
client at the binary and pass an access mode with `--mode` (omit it for read-only).

Claude Desktop, edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "tidesman": {
      "command": "/opt/homebrew/bin/tidesman",
      "args": ["--mode=safe"]
    }
  }
}
```

Claude Code:

```bash
claude mcp add --transport stdio --scope user tidesman \
  -- /opt/homebrew/bin/tidesman --mode=safe
```

Any stdio MCP client (OpenAI Codex and others) works the same way: point it at the binary and
pass the flags as arguments.

## What it does

Nine container tools, each tagged by what it can do (Read, Write, or Destructive):

| Tool | Capability | What it does |
|---|---|---|
| `system_ping` | Read | check the container service is reachable |
| `container_list` | Read | list containers |
| `container_inspect` | Read | show a container's full details |
| `container_logs` | Read | fetch a container's recent output |
| `container_run` | Write | create and start a container from an image |
| `container_exec` | Write | run a command inside a running container |
| `container_stop` | Write | gracefully stop a running container |
| `container_kill` | Write | stop a container by sending it a signal (SIGKILL by default) |
| `container_delete` | Destructive | remove a container |

What the capability tags mean: a **Read** tool observes state and changes nothing. A **Write**
tool changes a container's state but never removes it; the container survives and can be
inspected or restarted. **Destructive** is reserved for removing the container itself. Your MCP
client may separately mark some write tools (exec, stop, kill) as destructive, based on the MCP
hints each tool carries: those tools can end work or change data inside a container, even
though the container itself survives.

## Access modes: safe by default

Tidesman runs at one of three authority levels, set with `--mode=`, and it starts in the
safest one. If no `--mode` is given, it is read-only.

| Mode | Allows | Tools available |
|---|---|---|
| `read-only` (default) | Read | ping, list, inspect, logs (4) |
| `safe` | Read + Write | the above, plus run, exec, stop, kill (8) |
| `full` | Read + Write + Destructive | all nine, including delete |

A separate flag, `--allow-host-mounts`, is required before `container_run` may mount a folder
from your Mac into a container; it is off by default because a host mount reaches outside the
container's isolation onto your real files.

## Security and trust

- Signed and notarized. The binary is signed with an Apple Developer ID certificate and
  notarized by Apple, so Gatekeeper runs it without a warning.
- Safe by default. Unset mode is read-only; destructive operations require `full`; host mounts
  require an explicit flag.
- Audited. Every tool call is recorded to `~/Library/Logs/tidesman/audit.log` and the macOS
  unified log: its name, its arguments (with secret-like values redacted), and its outcome.
- Honest tool annotations. Every tool declares MCP read-only and destructive hints that match
  what it can actually touch, so your client knows the stakes and can ask before anything
  risky runs.

## Privacy Policy

Tidesman collects nothing about you and sends nothing to us: no telemetry, no analytics, no
crash reporting, no accounts. Everything it does happens on your Mac.

- The audit log (tool calls, arguments with secret-like values redacted, outcomes) is written
  only to your Mac (`~/Library/Logs/tidesman/audit.log` and the macOS unified log) and stays
  there unless you share it yourself. Delete it whenever you like.
- Tidesman opens no network ports. Its only outbound connection is to a container registry
  (such as Docker Hub) when you or your assistant asks it to pull an image; that request shares
  the image name and your network address with the registry, like any container tool.
- Nothing is shared with, sold to, or retained by anyone else.

The full policy lives at [tidesman.dev/privacy.html](https://tidesman.dev/privacy.html).
Privacy questions: [legal@tidesman.dev](mailto:legal@tidesman.dev).

## License

Tidesman is proprietary software, provided free of charge under the end-user license in
[EULA.txt](EULA.txt). It bundles open-source components whose notices are preserved in
[THIRD-PARTY-LICENSES](THIRD-PARTY-LICENSES).

## Links

- Website: https://tidesman.dev
- Report an issue: https://github.com/JeronimoColon/tidesman-mcp/issues
- Contact: hello@tidesman.dev
