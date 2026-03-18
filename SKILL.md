---
name: cpp-sandbox-debug
description: Deep C++ debugging workflow for sandboxed or tool-limited environments where GUI debuggers, unrestricted process inspection, or ad-hoc tracing tools may be unavailable. Use when debugging crashes, parse failures, launch failures, config/state mismatches, race conditions, timing bugs, audio/render glitches, or other local C++ repo issues through code inspection, focused repros, narrow instrumentation, and build/test validation.
---

# C++ Sandbox Debug

Debug deeply without assuming:
- GUI debugger access
- unrestricted process attach
- external tracing tools
- broad system visibility

Prefer this loop:
1. reproduce
2. localize
3. instrument narrowly
4. convert to a deterministic repro
5. fix conservatively
6. verify

## Core Rules

### Make the bug smaller before making theories bigger

Prefer:
- one file
- one input
- one config
- one failing target
- one code path

If the app is GUI-heavy, reduce it to one of:
- a unit test
- a parser/loader repro
- a minimal CLI path
- a deterministic fixture

### Debug boundaries, not the whole program

Inspect the boundary where truth should change shape:
- file parse -> in-memory model
- config load -> runtime settings
- menu or CLI selection -> launch options
- chart or asset load -> runtime object
- thread handoff -> queue payload or callback input
- device init -> actual runtime format/rate
- cache load -> filtered visible state

At each boundary, verify actual values instead of inferred intent.

### Instrument narrowly

Add temporary or gated instrumentation only where it collapses uncertainty.

Prefer:
- one high-signal warning with real paths, formats, counts, or IDs
- one assertion on an invariant-rich path
- one focused test fixture
- one one-shot diagnostic around the suspect boundary

Avoid:
- per-frame logs
- per-sample logs
- global tracing spam
- multi-layer logging before the failing layer is known

### Prefer deterministic artifacts

Best evidence, in order:
1. failing test
2. reproducible CLI or log sequence
3. one-shot targeted warning or assertion
4. user report without local repro

If no test exists, add one when feasible.

## Workflow

### 1. Triage

Collect:
- exact symptom
- exact failing input, file, or config
- latest relevant logs
- active build target and configuration
- regression window, if known

Classify the problem early:
- parse or format mismatch
- config or state mismatch
- path, unicode, or file-resolution bug
- timing or sample-rate bug
- thread, race, or lock-scope bug
- data transformation bug
- rendering-only or presentation bug

### 2. Reproduce

Use the smallest available loop.

Common starting commands:

```powershell
rg -n "needle" src tests
rg --files src tests
Get-Content path\to\file.cpp
cmake --build build --config Release --target <target>
ctest --test-dir build -C Release --output-on-failure -R <test_name>
```

If a GUI flow is broken:
- trace the launch path
- inspect the actual runtime object, not just menu state
- reproduce through a loader, parser, or test path if possible

### 3. Localize

Trace the values that cross the relevant boundary.

Examples:
- parser bug: compare the real input fields against parser expectations
- launch failure: compare selected object state against loaded profile or runtime overrides
- audio bug: compare requested sample rate, actual device rate, and source rate
- cache bug: compare scanned data, cached data, and filtered visible data

### 4. Add a focused repro

Prefer adding a small test under the repo's existing test structure when the subsystem allows it.

Good repros:
- one malformed or real-world input file
- one stale config combination
- one bad cache record
- one fixture proving lane-count, format, or state behavior

### 5. Fix conservatively

Fix the narrowest layer that owns the truth.

Use this heuristic:
- real-world file variance -> parser or loader
- stale state crossing subsystems -> boundary reconciliation
- unsupported external inputs -> early probe or filter
- runtime mismatch between selected object and config -> launch or session init

Do not push user-facing policy into unrelated low-level code if the boundary layer can resolve it cleanly.

### 6. Verify

Run:
- the smallest focused test first
- then the main target build
- then the broader suite, if available

If build outputs collide under MSBuild or antivirus file locking, rerun sequentially before treating the failure as a code regression.

## Debug Patterns

### Parser or real-world format bug

Use this pattern:
1. inspect the real input
2. compare field names, casing, delimiters, and encoding against parser assumptions
3. add compatibility handling or an early probe
4. add a regression test from the real-world pattern

### Config or state mismatch bug

Use this pattern:
1. inspect persisted config
2. inspect runtime overrides
3. inspect the final object handed to the failing subsystem
4. reconcile stale values at the ownership boundary if they should not hard-fail

### Threading or callback bug

Use this pattern:
1. find the handoff boundary
2. inspect queue payloads or state snapshots, not the entire hot loop
3. verify lock scope and ownership duration
4. confirm with timing-sensitive tests or narrow runtime diagnostics

### Audio or timing bug

Reason in:
- samples
- sample rate
- actual device rate
- source rate

Do not reason only in milliseconds if the pipeline consumes samples.

### Render or GUI-launch bug in a restricted environment

If direct GUI validation is weak:
- verify the data passed into the render model
- verify the state transition into the failing screen
- add tests for upstream view-model or filter logic
- leave one targeted runtime warning if a manual-only gap remains

## Repo Reconnaissance

Early in the session:
- find the entrypoint
- find the build target that exercises the bug
- find nearby tests
- find the first boundary where the wrong state becomes observable

Typical reconnaissance commands:

```powershell
rg -n "main\\(|WinMain\\(|TEST_CASE|doctest|gtest|catch2" src tests
rg -n "initialize|launch|load|parse|render|callback|queue" src
```

Prefer reading the narrowest set of files that can explain the failing path.

## Build and Test Discipline

Default order:

```powershell
cmake --build build --config Release --target <focused_target>
cmake --build build --config Release --target <main_app_target>
ctest --test-dir build -C Release --output-on-failure -R <focused_test>
ctest --test-dir build -C Release --output-on-failure
```

If the repo uses another build system, keep the same principle:
- build the smallest target first
- verify the affected app or library target
- run the narrowest relevant tests before the full suite

When adding temporary diagnostics:
- remove them after the root cause is confirmed, unless they are durable warnings
- keep durable warnings one-shot and high-signal

## Anti-Patterns

Do not:
- guess from the error string alone
- patch three layers at once
- add hot-loop logging without a boundary hypothesis
- trust defaults without checking persisted state
- stop after a theory without a build or test
- call a manual GUI repro "verified" if only the upstream model was checked

## Definition of Done

The bug is only done when all are true:
- root cause is identified at a concrete boundary
- the fix lands in the correct ownership layer
- at least one repro path is validated
- the relevant build succeeds
- the relevant tests pass, or the remaining manual-only gap is stated explicitly
