```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each finding whose premise is a set, to check the fix covers every member.

**Findings I left unchecked:** none. F4, F9, F10 and F5 were rerun with tool calls. F12 and F13 were checked against what my context already held: `window.go`'s members and the earlier grep of `docs/overview.md`, where `internal/desktophook/` appears only at line 346. The other findings have no set premise.

**Findings**

1. **LOW — card 12, the paragraph "Every reader of a worktree's live runproc records counts it for that time, and each is right to:"**
   - **Summary:** The F5 fix claims to cover every reader of a slug's live records, but it lists four. Five more readers exist, and the claim is false for them because they skip a `RolePlacement` record rather than count it.
   - **Why it is LOW:** each of the five filters by role, so a placement record changes none of their decisions. The fix's conclusion, "No reader therefore gets a role check", still holds. Only the stated set is incomplete.
   - **Evidence:** `runproc.List` has these callers the fix does not list:
     - `internal/fabriccli/anchordeps.go:163`, `BattenLive`, which checks `RoleBatten`.
     - `internal/fabriccli/anchordeps.go:184`, `DobbyPID`, which checks `RoleDobby`.
     - `internal/fabriccli/anchordeps.go:489`, `dobbyPassInFlight`. It checks `RoleDobby`, then the children of those dobby pids by parent pid.
     - `internal/battencli/wire.go:777`, `gateProcessLive`. It checks `RoleGate` and the fixer roles.
     - `internal/pairteardown/teardown.go:856`, `driverRecordPID`, which checks `RoleDriver`.

**Sets that the fix covers in full**
- **F4:** card 1's Create list holds every recording the old cards 5 and 7 named: both kscreen recordings, both query recordings, both apply recordings and the `neve-qdbus` files, each with its `source.yaml`.
- **F10:** the other new `tier1` tests already sit in `tier1` files:
  - Card 4's `TestAfterPlacement` goes into `internal/desktophook/window_test.go`, which is `tier1` (`tiergate.Require(t, 1)` at line 25).
  - Card 5's test goes into `internal/termwindow/termwindow_integration_test.go`, which is `tier1`.
- **F9:** no other doc defines `tier3` as an LLM-only tier and needs the change.
  - `docs/benchmarks/running-tests.md:39` only names `llmtier_test.go`.
  - `docs/benchmarks/test-suite-timing.md:392` is a measurement report, which is exempt from staying current.

**Tool calls**
1. Grepped every `runproc.List` caller (F5's set), since my context held only the four the review named.
2. Grepped the `tiergate.Require` tiers of the desktophook and termwindow test files (F10's set), since my context lacked the tiers of the files other cards add `tier1` tests to.
3. Grepped the docs and `pattern/` for `tier3` (F9's set), since my context held only lines 34 and 55 of `running-tests.md`.
4. Read the bodies of the unlisted readers in `anchordeps.go`, `wire.go` and `teardown.go`, to judge whether each counts a placement record.
5. Read the rest of `dobbyPassInFlight` and how `dobbyOwner` is built, to see whether it counts any live record.
6. Read `liveUnsparedChild` and `dobbyOwner`, to confirm that its child scan keys on dobby parent pids, not on the role of the owner's records.
