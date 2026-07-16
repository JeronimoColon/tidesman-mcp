# Changelog

Notable changes to Tidesman. This project follows [Semantic Versioning](https://semver.org).

## [0.2.0] - 2026-07-16

- Five new tools for container images, bringing the surface to fourteen: `image_list` and
  `image_inspect` (read), `image_pull` and `image_tag` (write, available in safe mode), and
  `image_delete` (destructive, full mode only). Your assistant can now find, inspect, pull,
  tag, and remove the images your containers run from.
- Host-folder mounts are now scoped to explicit roots. Pass
  `--allow-host-mounts=/path/one[,/path/two]`; a mount is allowed only when its real path,
  with symlinks resolved, sits under one of those folders. The old bare `--allow-host-mounts`,
  which allowed any path, is no longer accepted.
- Clearer, more consistent error messages across every tool, backed by the container engine's
  completed typed-error contract.
- Launched directly in a terminal with no MCP client attached, Tidesman now prints a one-line
  hint explaining what it is, instead of waiting silently.

## [0.1.3] - 2026-07-05

- Every tool is now advertised in every access mode; the mode controls which ones may run,
  and a tool the mode locks says so in its description and names the mode that unlocks it. An
  assistant can see the full set of container tools even in read-only mode, so it knows what
  Tidesman can do and how to ask for more.
- The server now sends usage guidance when a client connects (what it is, the access modes,
  and how the user raises the mode), and `system_ping` reports the modes, what each unlocks,
  and how to change them.
- Tool descriptions are more thorough, including a note that a container needs a long-running
  command if you want to run further commands inside it.

## [0.1.2] - 2026-07-05

- Every tool now carries a human-readable title and complete MCP annotations (read-only,
  destructive, idempotent, and open-world hints), so MCP clients can label tools clearly and
  warn before anything risky runs.
- The `.mcpb` bundle declares the privacy policy: https://tidesman.dev/privacy.html.
- The product description now says what was always true: Tidesman is free.

## [0.1.1] - 2026-07-02

Post-launch hardening release.

- Added `--help`, `--version`, and `-h`.
- More accurate engine-down detection: Tidesman corroborates whether the container engine is
  actually running before telling you to start it, so a normal command failure is no longer
  mislabeled as "the engine isn't responding."
- `container_kill` is now idempotent on an already-stopped container, and its result reports
  honestly whether a signal was delivered.
- A misspelled or unknown image now reports as a bad image name rather than a raw registry
  authorization error.
- `system_ping` reports the active access mode (read-only / safe / full).
- Clearer, better-targeted error messages for delete and run failures.
- Non-zero exit codes on launch and configuration errors.

## [0.1.0] - 2026-07-01

First public release.

- Nine container tools spanning the foundation and core lifecycle: `system_ping`,
  `container_list`, `container_inspect`, `container_logs`, `container_run`, `container_exec`,
  `container_stop`, `container_kill`, and `container_delete`.
- Three access modes, safe by default: `read-only` (the default), `safe`, and `full`.
- Host-folder mounts gated behind an explicit `--allow-host-mounts` flag.
- Audit logging of every tool call to `~/Library/Logs/tidesman/audit.log` and the macOS
  unified log, with secret-like values redacted.
- Distributed as a signed, notarized binary (zip, `.pkg`, and `.mcpb`) for Apple Silicon Macs
  running macOS 26.
