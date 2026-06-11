# Reviews — plan of plan

| Field | Value |
| --- | --- |
| Target | `plan` |
| Verb reviewed | `plan` |
| Rounds | 2 |

## Round 1

| Field | Value |
| --- | --- |
| ID | `review-plan-nml-replacement-platform-prerequisite-1` (#1197) |
| Status | `active` |
| Findings | 2 |
| Severity | major 2 |
| Finding status | fixed 2 |

### Findings

#### F1 [#1321] - major - fixed

- Location: `m1:T2 gate`
- Problem: The extraction task claims the full `os`, `rt`, `loop`, `wire`, and `ipc` module layouts are copied, including private RT internals, tests, and fuzz trees where present, but its gate checks only five representative headers and excluded-directory absence.
- Impact: An implementer could satisfy the gate with partial or stub module directories while omitting sources, private headers, tests, or fuzz inputs needed by later build and verification tasks. The plan would then advance past the actual extraction boundary without proving the prerequisite source set exists.
- Required Fix: Tighten the task gate to compare the selected source module file inventories against `/home/jan/projects/mce-platform` and fail if any required file is missing, while still checking excluded modules are absent.
- Resolution: Updated m1:T2 with an added acceptance check and a gate that compares every selected source module file inventory against the extracted destination module before checking excluded modules are absent.

#### F2 [#1322] - major - fixed

- Location: `m1:T3 and m2:T1 gates`
- Problem: The standalone build/test tasks require dependency resolution through pinned `cstl` and test coverage for the extracted modules, but the gates only configure/build or run CTest. They do not explicitly fail on lingering `/home/jan/projects/mce-v2` CMake references, and the test gate can pass with a reduced or empty registered test set if the build files accidentally omit tests.
- Impact: The plan could accept a standalone-looking build that still leaks the source `mce-v2` tree through CMake paths, or a passing CTest run that does not actually register the OS, RT, loop, wire, and IPC tests needed to de-risk LinuxCNC consumption.
- Required Fix: Update the CMake build gate to grep extracted CMake files for forbidden `mce-v2`/source-platform references after a successful build, and update the test gate to inspect `ctest -N` output for expected OS, RT, loop, wire, IPC, seqlock, and shared-memory tests before running the suite.
- Resolution: Updated m1:T3 to fail on retained mce-v2/source-platform CMake references and updated m2:T1 to inspect ctest -N output for representative OS, loop, wire/frame, IPC, seqlock, and shared-memory tests before running CTest.


## Round 2

| Field | Value |
| --- | --- |
| ID | `review-plan-nml-replacement-platform-prerequisite-2` (#1198) |
| Status | `active` |
| Findings | 0 |

_No findings._
