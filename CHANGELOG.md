# Changelog

Notable changes to Tidesman. This project follows [Semantic Versioning](https://semver.org).

## [0.1.0] - unreleased

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
