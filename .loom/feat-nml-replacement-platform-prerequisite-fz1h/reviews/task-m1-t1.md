# Reviews — task of m1:T1

| Field | Value |
| --- | --- |
| Target | `m1:T1` |
| Verb reviewed | `task` |
| Rounds | 2 |

## Round 1

| Field | Value |
| --- | --- |
| ID | `review-task-nml-replacement-platform-prerequisite-1` (#1202) |
| Status | `active` |
| Findings | 1 |
| Severity | major 1 |
| Finding status | fixed 1 |

### Findings

#### F1 [#1341] - major - fixed

- Location: `/home/jan/projects/mce-platform/CMakeLists.txt:27`
- Problem: `MCE_PLATFORM_CSTL_SOURCE_DIR` defaults to `/home/jan/projects/cstl` and the README says cstl is pinned, but the scaffold does not verify or record the expected cstl revision at configure time. Any checkout at that path satisfies the gate.
- Impact: The task acceptance allows a pinned local override, and the overall plan depends on a known cstl revision. Without a revision check, later extraction and downstream smoke tasks can silently build against a different cstl API than the one planned, weakening the reproducibility LinuxCNC needs before pinning the platform dependency.
- Required Fix: Add a CMake-time check for the expected cstl git revision or equivalent explicit pin metadata, and document the expected revision in the README so the local override is actually pinned.
- Resolution: Added CMake-time verification that the local cstl checkout is exactly 1c1e9dac9638e574fccc2ff5706b4f27eb381850, documented the pin in README.md, reran the m1:T1 build gate successfully, and committed the fix as af5808c.


## Round 2

| Field | Value |
| --- | --- |
| ID | `review-task-nml-replacement-platform-prerequisite-2` (#1203) |
| Status | `active` |
| Findings | 0 |

_No findings._
