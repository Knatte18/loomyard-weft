---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/shuttleengine/claudeengine/hookcmd_integration_test.go:TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly
    summary: the computed-rm hook test pins no rm reached through a separator, pipe or xargs, and no special-parameter pass-through, though the row comment and doc.go state both
    fix: In TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly add denied rows "cd x && rm $DIR/f" and "ls | xargs rm $F", and allowed rows "rm -f .scratch/a-$$" and "rm -f .scratch/a && echo $?", so the command-position reach and the unlisted special parameters the settings.go row comment and claudeengine doc.go describe are pinned at the hook.
---

### [LOW] Computed-rm hook test leaves two stated behaviors unpinned

**Location:** `internal/shuttleengine/claudeengine/hookcmd_integration_test.go`, `TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly`
**Issue:** The rm row's comment and `claudeengine/doc.go` state that `rm` is denied in command position (reached through `&&`, `;`, a pipe or `xargs`) and that `$$`, `$?`, `$#` and `$!` pass.
The test's rows all start with `rm`, and none uses a special parameter.
The `$$` pass-through is exercised only by the `llm`-tier smoke scenario `TestSmokeTurnsForkPermissionAsk`, which is not a deterministic pin.
The plan's card 4 lists exactly the rows present, so this is a coverage gap of the plan, not a deviation from it.

## Rejected

- The `fork_permission_request_is_an_ask_without_an_event` row in `signals_test.go` exercises the same parser branch as `permission_request_is_an_ask_without_an_event`, since the parser ignores `agent_id`; card 7 names the row explicitly, so it is not raised.
- The in-place deny falsely denies a `sed -n` read whose payload description holds a ` -i…` word, and `perl -MFile::Spec`; both fall under the bound the row comment and doc.go already state as a guardrail, and card 3 accepts them.
- Suffixed and long in-place spellings stay outside the no-denied-recovery token scan; card 5 specifies this and the test pins it.
- Cards 1, 2, 5, 6 and 7 match their plan text: `.scratch/` is excluded through the idempotent `ExcludeAnchoredDir`, the classifier regex admits only `p` commands behind line or `/RE/` addresses, the dialog notice fires once per episode after the 60 s bound for autonomous runs with a notifier, and the table-driven `TestWait_DialogNotice` pins the bound, the once-only state and the persisted offset.
- The `deny-row-shape` decision holds: the new steers and notices hold no quote, backslash or line break, both rows state a guardrail bound, and one `standingDenies` table still builds the hooks and the notice.
- `lyx gate test` passes for `claudeengine` and `fabricengine` (integration tag), `cmd/lyx`, `contracts/stencils`, `editdirective`, `shuttleengine`, `shuttlecli` and `websterengine`.
