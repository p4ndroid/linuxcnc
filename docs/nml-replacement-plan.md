# NML Replacement Plan With Platform Extraction Prerequisite

## Summary

Replace LinuxCNC's first NML use case with a lightweight local IPC design, but
do not start by changing LinuxCNC. First extract the reusable platform layer from
`/home/jan/projects/mce-v2/src/platform` into its own CMake repository, modeled
after `/home/jan/projects/cstl`.

The LinuxCNC work depends on that platform repository. The goal is to reuse the
existing IPC, wire framing, fd passing, shared-memory, and seqlock primitives
rather than rewriting queues and IPC inside LinuxCNC.

## Platform Prerequisite

Create a standalone platform repository, tentatively
`/home/jan/projects/mce-platform`.

The extraction should:

- Follow the `cstl` pattern: standalone CMake project, consumed by reference
  from downstream projects, with pinned revisions rather than vendored copies.
- Consume `/home/jan/projects/cstl` for the extracted core dependencies instead
  of depending on the full `mce-v2` tree.
- Refactor platform CMake links from in-tree `mcl::` / `mce::result` providers
  to the exported `cstl` targets and preserved aliases.
- Preserve public headers, symbol prefixes, and behavior where practical, so
  LinuxCNC can adopt the platform library without learning a new IPC API.
- Keep the initial extracted scope to the modules LinuxCNC needs:
  `os`, `rt`, `loop`, `wire`, and `ipc`.
- Defer `drm`, `wayland`, generic UI `platform`, and unrelated graphics/input
  backend work unless a later consumer needs them.

The platform extraction is a separate plan and should be completed, tested, and
pinned before LinuxCNC starts consuming it.

## LinuxCNC NML Direction

Keep the first LinuxCNC replacement local and AXIS-focused.

The first LinuxCNC pass should:

- Preserve the AXIS-facing Python API shape:
  `linuxcnc.stat()`, `linuxcnc.command()`, and `linuxcnc.error_channel()`.
- Replace only the top-level NML transport for `emcCommand`, `emcStatus`, and
  `emcError`.
- Keep the existing `EMC_*` command, status, and error structs as the first
  local ABI.
- Leave task-to-motion RTAPI shared memory unchanged.
- Leave HAL pins, `iocontrol.0`, and interpreter internal queues unchanged.
- Treat remote NML compatibility and legacy clients as later work.

The intended platform primitives are:

- `ipc` / Unix `SOCK_SEQPACKET` for command and error traffic.
- `wire` framing for typed local messages.
- fd passing for shared-memory setup.
- `rt` memfd shared memory plus seqlock for status snapshots.

## LinuxCNC Communication Model

The replacement should preserve the logical LinuxCNC channels while changing the
transport:

- `emcCommand`: AXIS sends typed `EMC_*` command payloads over local IPC. Command
  serial numbers remain the completion-correlation mechanism.
- `emcStatus`: task owns a shared-memory `EMC_STAT` snapshot protected by a
  process-shared seqlock. AXIS `stat.poll()` reads a consistent snapshot.
- `emcError`: task sends operator error/text/display messages over local IPC.
  AXIS `error_channel().poll()` drains received events.

`linuxcnctask` remains the owner of command execution and status publication.
The realtime motion path continues to use the existing RTAPI shared-memory
interface through `usrmotintf`.

## Test Plan

For the platform repository:

- Configure and build the standalone CMake project.
- Run `ipc`, `wire`, `rt`, `loop`, and `os` tests.
- Confirm the extracted repo links against `cstl` and not the full `mce-v2`
  tree.
- Confirm the exported targets are usable by a small downstream CMake smoke
  project.

For the LinuxCNC follow-up:

- Build `linuxcnctask`, the AXIS Python extension, and the new IPC bridge.
- Unit-test command send/receive, status seqlock reads, and error delivery.
- Run an AXIS simulator smoke test.
- Verify task-to-motion shared-memory behavior is unchanged.

## Assumptions

- `mce-platform` is the default new repo name unless renamed before execution.
- The platform extraction is completed and pinned before LinuxCNC consumes it.
- The first LinuxCNC target is local AXIS control only.
- Remote NML compatibility, `halui`, `linuxcncsh`, `emcrsh`, and other legacy
  clients are out of scope for the first pass.
- Lightweight binary local IPC is preferred over protobuf or any schema-heavy
  replacement.
