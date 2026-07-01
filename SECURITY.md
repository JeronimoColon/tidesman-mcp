# Security Policy

## Reporting a vulnerability

If you find a security issue in Tidesman, please report it privately to **hello@tidesman.dev**
rather than opening a public issue. Include the Tidesman version, your macOS version, and steps
to reproduce. You will get an acknowledgement, and a fix or mitigation will be prioritized.

## What Tidesman guards

Tidesman lets an AI assistant drive Apple's `container` tool, so its threat model is
first-class:

- It starts in read-only mode; write and destructive operations require an explicit `--mode`.
- It will not mount host folders into a container without the `--allow-host-mounts` flag.
- Every tool call is audited (with secret-like values redacted) to a file and the macOS unified
  log.
- The binary is signed with an Apple Developer ID certificate and notarized by Apple.

Apple runs each container in its own lightweight virtual machine, so commands inside a container
are isolated from your Mac. The controls Tidesman adds protect against destroying your resources
and against reaching out onto the host.
