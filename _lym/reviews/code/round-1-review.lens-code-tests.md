```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour: pinning test**

1. `run --model` takes `<alias>[<effort>]` or `<engine>:<model-id>[<effort>]` and picks the engine from the alias. Pinned by `TestRunCmd_ModelSpec` in `internal/shuttlecli/cli_test.go`. Its rows are `luna[high]` → codex, `sonnet` → claude, the escape form, no model (default engine), an unknown alias and a malformed spec. It asserts that only the expected engine's `Prepare` ran and checks the resolved `ModelChoice`.
2. The `--effort` flag on `run` is removed. No test asserts the flag is gone. The `EffortFlagSet` row was deleted. This is a removal, so I raise no finding.
3. A `model_not_found` `StopFailure` is marked `ModelRejected`.
   - Pinned by the `stop_failure_on_an_unknown_model_rejects_the_model` row in `claudeengine/signals_test.go`.
   - The neighbouring `rate_limit` row compares the whole struct, so it pins `ModelRejected` as false for other errors.
4. A rejected model ends the run `api-error` at once, with and without `APIErrorRetries`. Pinned by two new rows in `TestWait_APIErrorRetry` (`apierrorretry_test.go`), `retries: 0` and `retries: 2`. Both assert `OutcomeAPIError` and the error text.
5. A rejected model with every output file present ends `done`, and a covering pause holds the end. These are the two exceptions the new doc text states. The rejected path has no row for either exception. Existing rows cover them for ordinary API errors, but the rejected branch sits before the retry-count branch and takes its own `endOnAPIError` call. **No pinning test for either exception on that branch** (finding F1).
6. The run envelope reports `apiError` when `result.APIError` is non-empty. **None.** A grep of `internal/shuttlecli/*_test.go` finds no assertion on the `apiError` field (finding F2).
7. `OpenLaunch` returns the Codex command with no hooks, no first turn and an empty resume line.
   - Pinned by `TestOpenLaunch/BypassDefault` in `codexengine/open_test.go`, which compares the whole `Launch` with `reflect.DeepEqual`.
   - That also pins no `CreelHooks` and no `SessionID`.
   - It also asserts the trust record roots and that the session dir stays unmade.
8. `OpenLaunch` refuses a non-bypass permission mode, an unlisted effort and Codex rules in the worktree, and writes nothing. Pinned by the `PromptPermissionMode`, `EffortOutsideEfforts` and `RulesInWorktree` rows, which check the lym home is unmade. The version-pin refusal has no `OpenLaunch` row. It is `validateChoice` code shared with `Prepare` (see F3).
9. `codexCommand` leaves out `--dangerously-bypass-hook-trust` when there are no hooks, and leaves out the positional prompt when there is no first turn. Pinned by `TestBuildLaunchCmd`. The reworked `NoWritableRootModelOrEffort` row keeps the run line unchanged. The new `opened` assertion pins the no-hooks, no-first-turn, no-exit command.
10. An engine without `SessionOpener` is refused before anything is created. Pinned by the existing "an engine without the capability is refused" row in `shuttleengine/open_test.go:53`.
11. `Codex` implements `SessionOpener`. Pinned at compile time by the new `var _ shuttleengine.SessionOpener = (*Codex)(nil)` and by `TestOpenLaunch`.
12. The refusal's way forward no longer names the assist model key, and it states the "tracked or not" rule. Pinned by the `RulesInWorktree` row in `TestPrepare_RefusesBeforeWriting`, which asserts "remove the directory" and "launch on an engine other than codex". No test asserts that an untracked rules folder refuses. The fixture folder is created with `mkdirAll` and never git-added, so the row exercises exactly the untracked case. That covers it.
13. The Warn wording "holds a Codex config". Not pinned as text. I raise no finding, since log wording is prose.
14. `TestPrepare_AppendsStandingPassages` keeps the fixture's Codex home. Pinned by the new trust-roots assertion on `repo.lymHome` and by the board base layer written with `writeCodexConfig`.
15. The `--list-models` and help text naming which engines `open` can start is static prose, so it is exempt.

**Findings**

- F1, LOW, design. `internal/shuttleengine/apierrorretry_test.go`, the rejected-model rows. The rejected branch has no row for "every output file exists → `done`" or for "a pause covers the run → no end". Add both as rows with `REJECT:model_not_found` events. The evidence is the rejected branch at `apierrorretry.go` (`if watch.rejected { return run.endOnAPIError(...) }`), which has its own path and no row for either exception.
- F2, LOW, design. `internal/shuttlecli/run.go`, the `if result.APIError != ""` branch that adds `fields["apiError"]`. No CLI test asserts the envelope carries `apiError`, or omits it on other outcomes. The summary and the help text promise this field. Add a `runCmd` case that uses a runner returning an `OutcomeAPIError` result, or a unit test of the field assembly.
- F3, NIT, design (test economy). `codexengine/open_test.go`, rows `PromptPermissionMode`, `EffortOutsideEfforts` and `RulesInWorktree`. They re-pin the refusals of `validateChoice` and `readyHome`. `TestPrepare_RefusesBeforeWriting` already covers `validateChoice` and the `RulesInWorktree` refusal in `readyHome`. Each `OpenLaunch` row does assert the extra fact "unmade lym home" through the public `OpenLaunch` surface. The overlap is partial, and I'd trim a row only if the maintainers want strict economy.

**Tool calls (1)**
- `grep` for `apiError|APIError` in `internal/shuttlecli/*_test.go`, plus `cannot open a session` in the tests. My context lacked the full test inventory, and it confirmed F2 and behaviour 10.
