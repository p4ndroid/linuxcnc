# Plan: nml-replacement-platform-prerequisite

| Field | Value |
| --- | --- |
| ID | `plan-nml-replacement-platform-prerequisite` (#1194) |
| Kind | `plan` |
| Status | `active` |
| Links: design | `design.json` |

## Prerequisites

- The implementer must have write permission for `/home/jan/projects/mce-platform`, which is outside the current LinuxCNC checkout.
- The build environment must be available at `/home/jan/projects/build-env/devenv.sh`, matching the `cstl` build instructions that require `$BUILD_ENV/cmake`.
- `/home/jan/projects/cstl` must be present and pinned; the inspected revision during planning was `1c1e9dac9638e574fccc2ff5706b4f27eb381850`.
- The source platform tree used for extraction is `/home/jan/projects/mce-v2/src/platform`; the inspected `mce-v2` revision during planning was `03c5a5c8e94a48ab60f62de567232ca00a01d3b8`.

## Milestones

### m1 — Standalone Platform Repository

- **m1:T1** [#488] (todo) — Scaffold `/home/jan/projects/mce-platform` with cstl-style CMake
  - reads: `docs/nml-replacement-plan.md`
  - reads: `.loom/feat-nml-replacement-platform-prerequisite-fz1h/design.md`
  - reads: `/home/jan/projects/cstl/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/README.md`
  - reads: `/home/jan/projects/mce-v2/src/platform/AGENTS.md`
  - reads: `/home/jan/projects/mce-v2/src/platform/CMakeLists.txt`
  - acceptance: `/home/jan/projects/mce-platform` exists as a standalone repository skeleton with a top-level `CMakeLists.txt`, `README.md`, and module-directory placeholders for the planned extraction.
  - acceptance: The top-level CMake follows the `cstl` pattern: CMake 4.0 project, `include(CTest)`, warnings/sanitizer/fuzzing options, `$BUILD_ENV/cmake` helper module loading, and `Threads` discovery where needed.
  - acceptance: The repository consumes `/home/jan/projects/cstl` by pinned reference or pinned local override and does not add `/home/jan/projects/mce-v2` to `CMAKE_MODULE_PATH`, include paths, or subdirectories.
  - acceptance: The initial configure/build succeeds with tests disabled before platform sources are copied in.
  - Gate: kind=build; command=`bash -lc 'source /home/jan/projects/build-env/devenv.sh && export CC="$BUILD_ENV/bin/gcc" CXX="$BUILD_ENV/bin/g++" && cmake -S /home/jan/projects/mce-platform -B /home/jan/projects/mce-platform/build -G Ninja -DBUILD_TESTING=OFF && cmake --build /home/jan/projects/mce-platform/build'`
    - test: `standalone configure`
    - test: `standalone build with BUILD_TESTING=OFF`
- **m1:T2** [#489] (todo) — Extract the five prerequisite modules into the new repository
  - reads: `/home/jan/projects/mce-v2/src/platform/os`
  - reads: `/home/jan/projects/mce-v2/src/platform/rt`
  - reads: `/home/jan/projects/mce-v2/src/platform/loop`
  - reads: `/home/jan/projects/mce-v2/src/platform/wire`
  - reads: `/home/jan/projects/mce-v2/src/platform/ipc`
  - reads: `/home/jan/projects/mce-v2/src/platform/CMakeLists.txt`
  - reads: `/home/jan/projects/mce-v2/src/platform/AGENTS.md`
  - acceptance: The new repository contains only the initial platform modules `os`, `rt`, `loop`, `wire`, and `ipc`, with each module retaining its `include/mce`, `src`, `test`, and `fuzz` layout where present.
  - acceptance: Public headers for the five modules remain under `include/mce/...` with the same filenames as the source tree.
  - acceptance: Private RT internals under `rt/src/internal` and `rt/src/alloc/internal` are present so RT allocation, capability, deadline, resource-limit, and task-init code can build from the extracted tree.
  - acceptance: Excluded modules `fs`, generic `platform`, `wayland`, `drm`, graphics, input, and Vulkan backend code are not present in `/home/jan/projects/mce-platform`.
  - acceptance: The extraction inventory check compares every file under each selected source module against the destination module, so missing private sources, tests, or fuzz files fail the task gate.
  - depends_on: `m1:T1`
  - Gate: kind=integration; command=`bash -lc 'for d in os rt loop wire ipc; do test -d "/home/jan/projects/mce-platform/$d" && (cd "/home/jan/projects/mce-v2/src/platform/$d" && find . -type f | sort) > "/tmp/mce-platform-src-$d.files" && (cd "/home/jan/projects/mce-platform/$d" && find . -type f | sort) > "/tmp/mce-platform-dst-$d.files" && diff -u "/tmp/mce-platform-src-$d.files" "/tmp/mce-platform-dst-$d.files"; done && test ! -e /home/jan/projects/mce-platform/fs && test ! -e /home/jan/projects/mce-platform/platform && test ! -e /home/jan/projects/mce-platform/wayland && test ! -e /home/jan/projects/mce-platform/drm'`
    - test: `full selected-module file inventory matches source modules`
    - test: `excluded-module absence`
- **m1:T3** [#490] (todo) — Rewire module CMake links to pinned cstl and preserved platform aliases
  - reads: `/home/jan/projects/mce-platform/CMakeLists.txt`
  - reads: `/home/jan/projects/mce-platform/os/CMakeLists.txt`
  - reads: `/home/jan/projects/mce-platform/rt/CMakeLists.txt`
  - reads: `/home/jan/projects/mce-platform/loop/CMakeLists.txt`
  - reads: `/home/jan/projects/mce-platform/wire/CMakeLists.txt`
  - reads: `/home/jan/projects/mce-platform/ipc/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/result/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/mem/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/alloc/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/container/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/intern/CMakeLists.txt`
  - reads: `/home/jan/projects/cstl/io/CMakeLists.txt`
  - acceptance: `mce::os`, `mce::rt`, `mce::loop`, `mce::wire`, and `mce::ipc` are exported or aliased from the standalone repository and remain the names downstream consumers link against.
  - acceptance: The extracted modules link to `mce::result`, `mcl::alloc`, `mcl::mem`, `mcl::str`, `mcl::math`, `mcl::intern`, `mcl::container`, and `mcl::io` from the pinned `cstl` dependency rather than from any `mce-v2` source path.
  - acceptance: The module dependency order is buildable: `os` and `wire` are available before dependents, `rt` links `os` and cstl modules, `loop` links `os` and `rt`, and `ipc` links `loop`, `os`, and `wire`.
  - acceptance: A clean standalone configure/build with tests disabled succeeds without `/home/jan/projects/mce-v2` being present in the platform build graph.
  - acceptance: The build gate fails if extracted CMake files retain `/home/jan/projects/mce-v2`, `mce-v2/src/platform`, or source-platform-relative references that would make the standalone repository depend on the old tree.
  - depends_on: `m1:T2`
  - Gate: kind=build; command=`bash -lc 'source /home/jan/projects/build-env/devenv.sh && export CC="$BUILD_ENV/bin/gcc" CXX="$BUILD_ENV/bin/g++" && rm -rf /home/jan/projects/mce-platform/build && cmake -S /home/jan/projects/mce-platform -B /home/jan/projects/mce-platform/build -G Ninja -DBUILD_TESTING=OFF -DENABLE_WARNINGS=ON && cmake --build /home/jan/projects/mce-platform/build && ! grep -R -E "(/home/jan/projects/mce-v2|mce-v2/src/platform|CMAKE_SOURCE_DIR.*/src/platform)" /home/jan/projects/mce-platform/CMakeLists.txt /home/jan/projects/mce-platform/os/CMakeLists.txt /home/jan/projects/mce-platform/rt/CMakeLists.txt /home/jan/projects/mce-platform/loop/CMakeLists.txt /home/jan/projects/mce-platform/wire/CMakeLists.txt /home/jan/projects/mce-platform/ipc/CMakeLists.txt'`
    - test: `clean standalone configure`
    - test: `clean standalone build`
    - test: `cstl-backed target resolution`
    - test: `no extracted CMake references to mce-v2 source platform`

### m2 — Verification And Downstream Consumption

- **m2:T1** [#491] (todo) — Port extracted module tests and make the standalone test suite pass
  - reads: `/home/jan/projects/mce-platform/os/test`
  - reads: `/home/jan/projects/mce-platform/rt/test`
  - reads: `/home/jan/projects/mce-platform/loop/test`
  - reads: `/home/jan/projects/mce-platform/wire/test`
  - reads: `/home/jan/projects/mce-platform/ipc/test`
  - reads: `/home/jan/projects/mce-v2/src/platform/os/test`
  - reads: `/home/jan/projects/mce-v2/src/platform/rt/test`
  - reads: `/home/jan/projects/mce-v2/src/platform/loop/test`
  - reads: `/home/jan/projects/mce-v2/src/platform/wire/test`
  - reads: `/home/jan/projects/mce-v2/src/platform/ipc/test`
  - reads: `/home/jan/projects/mce-v2/src/platform/AGENTS.md`
  - acceptance: The standalone repository builds with `BUILD_TESTING=ON` and registers tests for OS duration/process/rand/thread/time behavior, loop behavior, wire/frame behavior, IPC server behavior, RT memory/task/sync/timing behavior, seqlock behavior, and shared-memory entity behavior.
  - acceptance: Tests that require source-tree private headers or helpers use paths within `/home/jan/projects/mce-platform`, not `/home/jan/projects/mce-v2`.
  - acceptance: IPC tests exercise command/error-style local socket behavior including server hardening/stress/soak coverage and fd-passing helpers where present.
  - acceptance: RT tests preserve the documented init-phase versus RT-phase constraints and cover seqlock/shared-memory behavior needed for LinuxCNC status snapshots.
  - acceptance: All registered standalone tests pass under CTest.
  - acceptance: The test gate checks `ctest -N` output for representative registered tests before running the suite, so a reduced or empty test set cannot satisfy the task.
  - depends_on: `m1:T3`
  - Gate: kind=unit; command=`bash -lc 'source /home/jan/projects/build-env/devenv.sh && export CC="$BUILD_ENV/bin/gcc" CXX="$BUILD_ENV/bin/g++" && rm -rf /home/jan/projects/mce-platform/build && cmake -S /home/jan/projects/mce-platform -B /home/jan/projects/mce-platform/build -G Ninja -DBUILD_TESTING=ON -DENABLE_WARNINGS=ON && cmake --build /home/jan/projects/mce-platform/build && ctest --test-dir /home/jan/projects/mce-platform/build -N > /tmp/mce-platform-ctest-list && grep -E "duration_test|os.*duration" /tmp/mce-platform-ctest-list && grep -E "loop_test|loop" /tmp/mce-platform-ctest-list && grep -E "wire_test|frame_test|wire|frame" /tmp/mce-platform-ctest-list && grep -E "server_test|ipc.*server" /tmp/mce-platform-ctest-list && grep -E "seqlock_test|seqlock" /tmp/mce-platform-ctest-list && grep -E "shmem_entity_test|shared|shmem" /tmp/mce-platform-ctest-list && ctest --test-dir /home/jan/projects/mce-platform/build --output-on-failure'`
    - test: `registered OS tests`
    - test: `registered RT tests`
    - test: `registered loop tests`
    - test: `registered wire/frame tests`
    - test: `registered IPC tests`
    - test: `registered seqlock and shared-memory tests`
    - test: `full CTest run`
- **m2:T2** [#492] (todo) — Add downstream CMake smoke consumption for cstl plus mce-platform
  - reads: `/home/jan/projects/mce-platform/CMakeLists.txt`
  - reads: `/home/jan/projects/mce-platform/README.md`
  - reads: `/home/jan/projects/cstl/README.md`
  - reads: `/home/jan/projects/cstl/CMakeLists.txt`
  - reads: `.loom/feat-nml-replacement-platform-prerequisite-fz1h/design.md`
  - acceptance: The repository contains or documents a smoke consumer that brings in pinned `cstl` and the standalone `mce-platform` repository by reference, not by copying sources.
  - acceptance: The smoke consumer links against exported platform targets such as `mce::ipc`, `mce::wire`, `mce::rt`, and `mce::os`.
  - acceptance: The smoke program includes representative headers for IPC, wire framing, RT/seqlock or shared memory, and OS utilities from `include/mce/...`.
  - acceptance: The smoke configure/build works from a clean directory outside `/home/jan/projects/mce-platform`, proving downstream consumers do not depend on build-tree-relative paths or the full `mce-v2` tree.
  - depends_on: `m2:T1`
  - Gate: kind=integration; command=`bash -lc 'source /home/jan/projects/build-env/devenv.sh && export CC="$BUILD_ENV/bin/gcc" CXX="$BUILD_ENV/bin/g++" && rm -rf /tmp/mce-platform-smoke-build && cmake -S /home/jan/projects/mce-platform/test/downstream-smoke -B /tmp/mce-platform-smoke-build -G Ninja -DMCE_PLATFORM_SOURCE_DIR=/home/jan/projects/mce-platform -DCSTL_SOURCE_DIR=/home/jan/projects/cstl && cmake --build /tmp/mce-platform-smoke-build'`
    - test: `downstream configure outside source tree`
    - test: `downstream build links exported platform targets`
- **m2:T3** [#493] (todo) — Record the pinned platform revision and LinuxCNC handoff
  - reads: `docs/nml-replacement-plan.md`
  - reads: `.loom/feat-nml-replacement-platform-prerequisite-fz1h/design.md`
  - reads: `/home/jan/projects/mce-platform/README.md`
  - reads: `/home/jan/projects/mce-platform/CMakeLists.txt`
  - acceptance: `/home/jan/projects/mce-platform` has a concrete git revision that downstream work can pin.
  - acceptance: The platform README or handoff document states the initial extracted module set, the excluded modules, the `cstl` pin, and the exact `mce-platform` revision intended for LinuxCNC consumption.
  - acceptance: The handoff explicitly says LinuxCNC NML replacement work may begin only after the standalone build, module tests, and downstream smoke gates have passed.
  - acceptance: The handoff maps the later LinuxCNC needs to platform capabilities: IPC for command/error channels, wire framing for typed payloads, fd passing for shared-memory setup, and RT shared memory plus seqlock for status snapshots.
  - depends_on: `m2:T2`
  - Gate: kind=integration; command=`bash -lc 'git -C /home/jan/projects/mce-platform rev-parse HEAD && grep -R "cstl" /home/jan/projects/mce-platform/README.md /home/jan/projects/mce-platform/docs 2>/dev/null && grep -R "LinuxCNC" /home/jan/projects/mce-platform/README.md /home/jan/projects/mce-platform/docs 2>/dev/null && grep -R "seqlock" /home/jan/projects/mce-platform/README.md /home/jan/projects/mce-platform/docs 2>/dev/null'`
    - test: `pinned revision exists`
    - test: `handoff mentions cstl`
    - test: `handoff mentions LinuxCNC`
    - test: `handoff mentions seqlock status capability`


## Deferred

- Any LinuxCNC source changes, AXIS bridge work, or NML transport replacement.
- Remote NML compatibility and legacy non-AXIS clients such as `halui`, `linuxcncsh`, and `emcrsh`.
- Extraction of `fs`, generic UI `platform`, Wayland, DRM, graphics, input, and Vulkan/backend code.
- API redesigns, public symbol renames, header-root changes, protobuf/schema-heavy transports, or vendoring platform sources into LinuxCNC.
