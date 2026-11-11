```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the adopt check mark and the reed attach/up refusal. Both behave correctly on every input I ran, so I have no findings.

**Cases run**

- `checkMarkOf`, green record: returns `checkMark{}`. This is the zero mark, so it carries no state or text.
- `checkMarkOf`, failed record with no tiers: "failed before any tier ended" text. Correct.
- `checkMarkOf`, failed record with `FailedTier` set (tier1): "failed at tier1; lym beam status shows why". Correct.
- `checkMarkOf`, failed record with tiers but `FailedTier` nil: generic "failed; lym beam status shows why". Correct.
- `checkMarkOf`, running record and the starting record (no tiers, `Running` nil): unchanged from before. The green-only branch that formatted tiers without outcomes is gone, and `parts` is built the same for the running case.
- `primeMark`, zero own mark, no other check running: `texts` stays empty, so the result is `{State: "", Text: "", Start: zero}`. `SetCheckMark` sees the empty text and unsets all three tmux options (`internal/reedengine/checkmark.go:24-30`). This clears the bar.
- `primeMark`, zero own mark, another subject running: only the other subject's text shows, with state running and a zero `Start`. The other subject's text from `checkMarkOf(record, time.Time{})` carries no timer, so losing the own `Start` costs nothing.
- `primeMark`, own failed plus another running: the joined text shows with state running. The failed item rides under a running state. That behavior is unchanged by the diff.
- `primeMark`, scratch directory unreadable: returns `own` unchanged, so a green zero mark still clears. Correct.
- `primeMark`, an own failed mark followed by a green re-run of the same subject: the green end sets the zero mark, which clears the stale failed text. Correct.
- Pre-run guard in `internal/reedcli/cli.go:166-172`, `attach`: `cmd.Name()` is "attach", so the `lym a` alias is also caught. Refused in the Bolt checkout and in a records sibling, with exit 1, before `LoadConfig` and before any tmux call. No state is left on the failure path, because `c.eng` is never set.
- Pre-run guard, `up`: same behavior as `attach`.
- Pre-run guard, the prime or an ordinary worktree: `RequireWarpWorktree` returns nil (`internal/fabricengine/fabric.go:116-128`), so the code falls through to the config load. Correct.
- Pre-run guard, other verbs such as `down`, `list`, `status`: not guarded, so `down` still removes a session an older binary booted in such a checkout.
- Error text: `output.Err` plus `clihelp.Abort(ctx, 1)` plus `return nil`, the same pattern as the neighboring branches. The message names the verb, fabric's reason, and the hub path `location.HubPath` as the way forward.

**Tool calls (2)**

1. `internal/adoptcli/check.go` 296-396: my context had only the diff hunks, not the full `checkMarkOf` and `primeMark` bodies.
2. `internal/adoptcli/check.go` 470-520: I needed the `Mark` callback that feeds `primeMark` and `SetCheckMark`, and my context lacked it.
