# WiSHbox / GameHubIOS Roadmap

## Project Summary

`GameHubIOS` is an iOS application that embeds:

- a native iOS shell/UI
- `Box64` as the x86_64 Linux userspace execution engine
- `iSH` components as the virtual filesystem and low-level Linux environment bridge

Planned product name:

- `WiSHbox`

Current reality:

- the app still uses the working name `GameHubIOS`
- the technical core is already capable of importing and running an external `x86_64 ELF` on an iPhone in `Box64` interpreter mode

## Long-Term Goal

Run Linux `x86_64` executables on iPhone through Box64, with a practical iOS UI, stable process/runtime behavior, and a path toward more complex applications after initial bring-up.

## Current Milestone

`M1: Stable external static ELF e2e in interpreter mode`

This milestone is achieved when:

- the app launches reliably
- the user can import an external `.elf` file from Files
- Box64 can initialize end-to-end on iPhone
- the ELF is loaded, mapped, and executed
- guest stdout is visible inside the app
- the guest exits cleanly
- the iOS app stays alive after execution

Current status:

- achieved

## Timeline So Far

### 1. iOS App Foundation

Started with building the base iOS application shell:

- app structure
- tab UI
- import/debug flows
- runtime bridge surface

Initial app name:

- `GameHubIOS`

Planned rename:

- `WiSHbox`

### 2. Box64 iPhone Bring-up

Created and adapted the Box64 side for iPhone bring-up:

- integrated Box64 into the iOS app build
- forced interpreter-mode during early bring-up
- deferred dynarec/JIT work until correctness was achieved

### 3. iSH Bridge Integration

Bridged Box64 to the iSH-backed environment:

- VFS initialization
- root mount
- first process bootstrap
- procfs/devpts setup

This made it possible for Box64 to execute inside an iOS sandbox with a Linux-like filesystem view.

### 4. Early Runtime Crash Fixing

Worked through the entire early crash/stall line in Box64 bring-up:

- bridge pages / `NewBridge`
- cycle log issues
- `__curbrk`
- `dlsym` null-path cases
- `MAP_FAILED` / null dereferences
- custom allocator bootstrap
- rbtree/custommem initialization
- wrapper sentinel / invalid function pointer paths
- alternate-map lazy-init crashes
- startup W^X / JIT-related crash paths

### 5. Post-Startup Stall Debugging

Once startup crashes were gone, the next blocker became a hang during `initialize()`.

This was debugged step-by-step by slicing execution with logs through:

- `addLibPaths`
- `ResolveFile`
- environment handling
- preload parsing
- argv/env setup
- file checks
- initial ELF open/load

### 6. ELF Loader Bring-up on iOS

The ELF loading path was then stabilized for iOS constraints:

- host-visible ELF paths used for initial open
- problematic `realpath` paths bypassed on Apple
- low-address fixed mapping fallback added
- relocation path introduced for main image load
- anonymous `PT_LOAD` mapping used on Apple where file-backed behavior caused failures

### 7. Sandbox and Exit Stability

The app-side stability line was fixed after Box64 started running:

- exit handling captured so guest `exit(0)` no longer killed the app
- post-exit cleanup softened for iOS bring-up
- dangerous process-creation paths (`fork/clone/vfork`) were blocked to avoid iOS sandbox termination
- post-execution app survival was confirmed

### 8. First Real Guest Execution

After runtime and loader stabilization:

- guest entrypoint resolution worked
- `DynaRun` was reached in interpreter mode
- an imported external `x86_64 ELF` executed on iPhone
- guest stdout was captured inside the app

Confirmed demo result:

- `Hello from Box64 on iOS!`

## What Is Already Working

### App-Level

- iOS app launches normally
- app no longer crashes during Box64 demo execution
- debug/demo flow can be manually triggered
- external ELF import works through Files

### Runtime-Level

- Box64 enters interpreter mode on iPhone
- VFS bootstrap succeeds
- `NewBox64Context` succeeds
- helper initialization succeeds
- `initialize()` completes
- ELF loads successfully
- entrypoint is resolved
- `DynaRun` executes guest code
- guest `exit(0)` returns cleanly

### Demo-Level

- user imports an external ELF manually
- selected ELF can be run from the `Debug` tab
- guest output is visible in a focused modal runner
- detailed logs remain available for debugging

## Where We Are Now

We are no longer in the "can Box64 even start?" phase.

We are now in the phase:

`From first successful static ELF execution -> toward broader executable compatibility`

In practical terms:

- the first external static ELF e2e path is done
- the next step is proving this is not a one-file fluke

## Roadmap Ahead

## M0. App Shell and Integration

Status:

- done

Includes:

- app creation
- UI shell
- Box64 integration
- iSH bridge wiring

## M1. First External Static ELF Execution

Status:

- done

Includes:

- manual ELF import
- ELF load on device
- Box64 interpreter execution
- guest stdout
- clean guest exit
- app survives post-exit

## M2. Static ELF Compatibility Suite

Status:

- next

Goal:

prove that the interpreter-mode path works for a small class of external static ELF executables, not just one demo file.

Suggested test binaries:

- `hello`
- `argc/argv`
- `env`
- `open/read/write`
- `brk/mmap`
- `clock/time`

Expected result:

- a compatibility table for "simple static x86_64 ELF on iPhone via Box64"

## M3. Dynamic ELF Bring-up

Status:

- upcoming

Goal:

reach first successful dynamic ELF execution.

Main areas:

- `PT_INTERP`
- dynamic loader path
- `DT_NEEDED`
- dependency resolution
- relocation handling
- early dynamic libc path

Expected result:

- first dynamic ELF reaches meaningful execution, ideally to `main()` or near it

## M4. Syscall and Runtime Coverage Expansion

Status:

- upcoming

Goal:

support a broader set of real Linux userland behavior.

Areas:

- file syscalls
- memory syscalls
- timing syscalls
- random/entropy
- futex-related behavior
- thread-related runtime expectations

Expected result:

- more real-world command-line programs begin to run

## M5. Process Model Strategy Under iOS Sandbox

Status:

- upcoming, high importance

Goal:

define what process creation means in a platform where true `fork()` is not available.

Areas:

- blocking or stubbing unsupported process syscalls
- documenting process-model limitations
- understanding which Linux applications remain viable without real host fork

Expected result:

- a stable and explicit compatibility model instead of ad hoc failures

## M6. Dynamic Runtime Quality and Usability

Status:

- later

Goal:

make runtime behavior more robust and less bring-up-specific.

Areas:

- reduce temporary tracing
- improve compatibility reporting
- make loader and runtime diagnostics easier to read
- improve repeated-run reliability

## M7. Dynarec / JIT Return Path

Status:

- later, not current priority

Goal:

return to performance work only after correctness is stable.

Areas:

- `MAP_JIT`
- W^X handling
- debugger/JIT entitlement flows
- dynarec correctness on iOS

Expected result:

- faster execution path after the interpreter line is already trusted

## M8. Productization and Public Identity

Status:

- partially started, not finished

Goal:

turn the bring-up prototype into a coherent product.

Areas:

- rename `GameHubIOS` -> `WiSHbox`
- improve branding
- improve demo UX
- reduce internal-only naming
- document supported workflows
- prepare public repository presentation

## Rename Plan: GameHubIOS -> WiSHbox

Status:

- planned

Why:

- `GameHubIOS` was a practical bootstrap name
- `WiSHbox` better reflects the actual identity of the project

Suggested rename scope:

- display name
- tab labels and user-facing copy
- project/scheme names
- bundle naming if desired
- repository branding
- README and roadmap text

Suggested timing:

- after the current milestone is committed and documented
- before broader public presentation

## Definition of "First App Launch"

For this project, "first app launch" should not mean only that the iOS shell opens.

A meaningful first launch target is:

1. app opens on iPhone
2. user imports a Linux `x86_64 ELF`
3. Box64 initializes
4. ELF is executed
5. stdout is shown inside the app
6. app remains alive after execution

That target is now complete for the first static demo ELF.

## Immediate Next Steps

### 1. Lock the milestone

- commit current working state
- preserve latest demo IPA
- preserve demo ELF and test instructions

### 2. Build a static ELF test suite

- add multiple tiny validation binaries
- check which syscalls/runtime features already work
- document failures cleanly

### 3. Document the project publicly

- README
- roadmap
- current status
- limitations
- demo instructions

### 4. Begin dynamic ELF bring-up

- first target should be a very small dynamic executable
- treat this as the next major technical milestone

## One-Line Status

`WiSHbox / GameHubIOS has successfully reached external x86_64 static ELF execution on iPhone through Box64 interpreter mode, with clean guest output and no app crash.`
