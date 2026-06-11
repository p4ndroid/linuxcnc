# LinuxCNC Core Code Map

This tree is reduced to the POSIX `uspace` realtime backend and AXIS as the
machine-control GUI. The map below points to the remaining core runtime:
`src/rtapi/`, `src/hal/`, `src/emc/`, and `src/libnml/`.

## Top-Level Subsystems

- `src/rtapi/` provides the realtime abstraction used by HAL and motion. In
  this tree the live backend is POSIX uspace: `src/rtapi/uspace_rtapi_main.cc`,
  `src/rtapi/uspace_rtapi_app.cc`, `src/rtapi/uspace_posix.cc`, and
  `src/rtapi/uspace_ulapi.c`.
- `src/hal/` is the hardware abstraction layer. The core shared-memory model is
  in `src/hal/hal_lib.c` and `src/hal/hal_priv.h`; command-line control is in
  `src/hal/utils/halcmd.c` and `src/hal/utils/halcmd_commands.cc`.
- `src/emc/` is the machine controller. The main task loop is in
  `src/emc/task/emctaskmain.cc`, interpreter/task glue is in
  `src/emc/task/emctask.cc`, realtime motion is in `src/emc/motion/motion.c`,
  `src/emc/motion/command.c`, and `src/emc/motion/control.c`.
- `src/libnml/` implements the NML/CMS messaging substrate used between task,
  motion, IO/status, and UI processes. Start with `src/libnml/nml/nml.cc`,
  `src/libnml/nml/nml.hh`, `src/libnml/cms/cms.cc`, and
  `src/libnml/cms/cms.hh`.
- `src/emc/usr_intf/axis/` is the retained machine-control GUI. The AXIS Python
  entrypoint is `src/emc/usr_intf/axis/scripts/axis.py`; its Tcl assets live in
  `share/axis/tcl/`.

## Build And Runtime Entrypoints

- Configure selects the only supported realtime backend in `src/configure.ac`
  with `--with-realtime=uspace`.
- The top-level build graph is assembled in `src/Makefile`; subsystem build
  fragments include `src/rtapi/Submakefile`, `src/hal/Submakefile`,
  `src/emc/task/Submakefile`, `src/emc/motion/Submakefile`,
  `src/emc/usr_intf/axis/Submakefile`, and `src/libnml/Submakefile`.
- The launcher is generated from `scripts/linuxcnc.in`; realtime start/stop
  wrapping is generated from `scripts/realtime.in`.
- Runtime NML channel configuration is in `configs/common/linuxcnc.nml`, with
  larger and split-host variants in `configs/common/linuxcnc_big.nml`,
  `configs/common/server.nml`, and `configs/common/client.nml`.

## Process And Shared-Memory Flow

1. `scripts/linuxcnc.in` prepares the environment and starts the configured UI,
   task, IO, and realtime pieces.
2. `scripts/realtime.in` reads `rtapi.conf`, verifies the uspace runtime, and
   uses `rtapi_app` rather than loading RTOS kernel modules.
3. `src/rtapi/uspace_rtapi_main.cc` starts `rtapi_app`; loaded realtime modules
   use the POSIX backend in `src/rtapi/uspace_posix.cc`.
4. HAL components call `hal_init()` and allocate HAL shared memory through
   `src/hal/hal_lib.c`; HAL metadata and object layout are defined in
   `src/hal/hal_priv.h`.
5. Motion, task, and UI exchange commands and status through NML channels
   described by `configs/common/linuxcnc.nml` and typed in
   `src/emc/nml_intf/emc.hh` and `src/emc/nml_intf/emc_nml.hh`.

## Key Message Types And Boundaries

- EMC command/status/error message classes are declared in
  `src/emc/nml_intf/emc.hh`; channel setup helpers are in
  `src/emc/nml_intf/emc_nml.hh`.
- Interpreter command lists are represented by `src/emc/nml_intf/interpl.hh`
  and implemented in `src/emc/nml_intf/interpl.cc`.
- Generic NML message mechanics are in `src/libnml/nml/nmlmsg.hh`,
  `src/libnml/nml/cmd_msg.hh`, and `src/libnml/nml/stat_msg.hh`.
- CMS transport and buffer behavior lives under `src/libnml/cms/` and
  `src/libnml/buffer/`; OS shared-memory and semaphore wrappers are under
  `src/libnml/os_intf/`.

## HAL And RTAPI Boundary

- HAL owns pins, parameters, signals, functions, and component bookkeeping in
  `src/hal/hal_lib.c` and `src/hal/hal_priv.h`.
- RTAPI owns realtime-safe allocation, scheduling, threads, shared-memory
  primitives, and process/module lifecycle through `src/rtapi/rtapi.h`,
  `src/rtapi/rtapi_app.h`, `src/rtapi/uspace_common.h`, and
  `src/rtapi/uspace_rtapi_app.cc`.
- `loadrt` and `unloadrt` route through `rtapi_app` from
  `src/hal/utils/halcmd_commands.cc`; remote HAL control follows the same
  uspace path in `src/hal/utils/halrmt.c`.

## Interpreter, Task, And Motion Flow

1. AXIS sends operator commands through the EMC/NML interface from
   `src/emc/usr_intf/axis/scripts/axis.py`.
2. Task receives commands in `src/emc/task/emctaskmain.cc` and dispatches plan
   and machine-state work through `src/emc/task/emctask.cc`.
3. The RS274 interpreter is entered through `src/emc/rs274ngc/rs274ngc_interp.hh`
   and related implementation files such as `src/emc/rs274ngc/interp_read.cc`.
4. Task queues canonical/interpreter output through `src/emc/nml_intf/interpl.cc`
   and motion/task commands through the message classes in
   `src/emc/nml_intf/emc.hh`.
5. Realtime motion consumes commands in `src/emc/motion/command.c`, executes
   control work in `src/emc/motion/control.c`, and exposes the motion module
   lifecycle in `src/emc/motion/motion.c`.
6. Trajectory planning support is under `src/emc/tp/`; kinematics modules are
   under `src/emc/kinematics/`.

## Where To Start

- Startup/debugging: `scripts/linuxcnc.in`, `scripts/realtime.in`,
  `src/rtapi/uspace_rtapi_main.cc`.
- HAL loading and graph issues: `src/hal/utils/halcmd_commands.cc`,
  `src/hal/hal_lib.c`, `src/hal/hal_priv.h`.
- Motion behavior: `src/emc/motion/command.c`,
  `src/emc/motion/control.c`, `src/emc/motion/motion.c`.
- Interpreter behavior: `src/emc/task/emctask.cc`,
  `src/emc/rs274ngc/rs274ngc_interp.hh`, `src/emc/rs274ngc/interp_read.cc`.
- NML protocol questions: `configs/common/linuxcnc.nml`,
  `src/emc/nml_intf/emc.hh`, `src/libnml/nml/nml.cc`.
- AXIS-only UI behavior: `src/emc/usr_intf/axis/scripts/axis.py` and
  `share/axis/tcl/axis.tcl`.
