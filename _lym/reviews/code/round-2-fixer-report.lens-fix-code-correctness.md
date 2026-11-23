```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the N1 and N2 changes. One LOW finding.

**LOW — `runsTmuxTier` refuses an `--upto` call that would not run the tmux tier** (`internal/gatecli/run.go`, `runsTmuxTier`)
- `--upto N` with `N >= 2` is treated as listing the tmux tier. The verb's own contract (`doc.go`: "`--upto N` adds every declared tier numbered 0 through N, skipping a tier marked `manual`") means a hub with tier2 marked manual, or no tier2 declared, would never run it from that range.
- A run agent's `--upto 3` call with no target and no filter is then refused for a tier it would not start. This is a false-positive refusal, and the call is rarely made. The refusal would need hub config to rule it out, which `flagShapeRefusal` is documented not to read.
- I ran the other branches step by step and they hold:
  - `--tier 2` alone: refused.
  - `--tier 2 ./pkg`: passes.
  - `--tier 2 -- -run X`: passes.
  - No `runAgent`: passes.
  - `--tier 1` and default: pass.
  - `--tier 2 --all`: refused.
- Callers of `flagShapeRefusal` still hold:
  - `cappedCall` treats a refusal as not capped, so `runTest` refuses in-process while the strand env is still present.
  - The `redfirst.go` and `rungate.go` request builders leave `runAgent` false.
- Test sensitivity, with the defect put back:
  - Gating the Claude row on `!interactive` fails the Claude `interactive` case and the manifest `want` for an interactive spec.
  - Gating the Codex row on `autonomous` fails the Codex `SwitchedToolHooks` "Interactive" case.
  - Ignoring `AllowRawTmux` in either engine fails `AllowRawTmuxSpecHoldsNoTmuxScriptRow`, the Claude exemption case, and the Codex `RawTmuxExemption` row.
  - Dropping the gate refusal fails every refused row of `TestFlagShapeRefusal_RunAgentTmuxTier`.
- No other file outside the changed packages consumes the renamed switch, so nothing else is left calling `SwitchAutonomous`. Only `configcli` and an `orchcli` test reference `SwitchDenyPython`, which is unchanged.

Tool calls:
1. `grep` for other users of `shellguard.Switch`, `buildDenyNotice` and `testRequest{`: my context lacked a repo-wide search for stale callers outside the engine packages.
