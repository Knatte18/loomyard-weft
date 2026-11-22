```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the goal point that asks for it, and each goal point to the changes that build it.

**Goal points** (board entry `## Goal` → `## Fix`; the record has no `Added after Discussion` entries):
- G1: `remote_control` defaults to `false` in the orch template.
- G2: hubs with no explicit value take the new default, and the change ships the reinterpretation `PATTERN-config-layers` asks for.
- G3: the RC button of a pane stays the operator's way to turn it on for one agent.
- G4: check that no other launch path turns remote control on by its own default.

**Changes → goal points:**
- C1 `internal/orchengine/template.yaml`: `remote_control: false`, comment adds "off unless the operator sets it". Traces to G1.
- C2 `internal/orchengine/reinterpret.go` (new) and the orch row in `internal/configreg/configreg.go:161`: `RemoteControlOffReinterpretationID` and `FlagRemoteControlDefault`, registered on the orch module. Traces to G2.
- C3 `internal/orchengine/doc.go:69-72`: the remote-control paragraph states the new default, the RC button and the reinterpretation. Forced by C1 and C2, since the doc describes the default the paragraph covers.
- C4 `internal/orchengine/config_test.go:223-226`: rows re-pinned to the template being off, with explicit `true` for the on cases. Forced by C1.
- C5 `internal/orchengine/reinterpret_test.go` (new): tests the new function. Forced by C2.
- C6 commit 5ccddabc ("break wrapped remote_control comments at sentence boundaries"): rewraps comments added by C1, C2 and C3. Forced by those changes.

**Goal points → changes:**
- G1 → C1, with C4.
- G2 → C2, with C5. The rewrite flags and does not rewrite, so a hub with no value takes the new default, as G2 asks. The flag is the way forward the PATTERN rule allows.
- G3 → no code change is needed. The RC path forms the name whatever the switch says (`internal/orchcli/remotecontrol.go:104`, `:154`). C3's doc line and C2's notice text state it. Built by leaving that path unchanged.
- G4 → this is a check, so no diff is required. The one spec field `RemoteControlName` is set only at `internal/orchcli/start.go:242`, fed from `launchRemoteControlName` (`start.go:784`). That function returns `""` while `cfg.RemoteControl` is false (`internal/orchcli/remotecontrol.go:71`), and no other grep hit sets `RemoteControlName`. The check's result is recorded in `_lym/landing/summary.md` under "Other launch paths".

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point is built or settled.

Tool calls: none.
