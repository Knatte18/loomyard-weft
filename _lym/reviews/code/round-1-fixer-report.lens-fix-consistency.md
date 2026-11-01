```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F8 change set.

**Findings**

1. **MEDIUM**: the F1 fix left twins of the same `-tags` smoke commands in two other crucible files.
   - Changed text: `crucible/orchestrator-prompt.md:119-124`, now `LYM_TEST_TIER=2 go test …` with `export LYM_TEST_TIER=2` before the loop.
   - Stale twins:
     - `crucible/review-prompt-template.md:135,149,150` still say `-tags tier2` / `-tags tier3`.
     - `crucible/README.md:132-134,144` still say `-tags tier2` / `-tags tier3` and `go test -c -tags tier2`.
   - Evidence: under the run-time selector these commands run tier0 only and skip every smoke test green. That is the silent skip F1 names, and this fix closed it in only one of three files. The review's `location` named the orchestrator prompt only, but its premise covers the whole set of crucible smoke commands.
   - The summary.md bullet "Left as they are: `crucible/`'s smoke commands" was removed. That is now wrong for these two files unless they are fixed too.

2. **LOW**: `pattern/PATTERN-test-speed.md:167` still says "a tier-tagged test with a call the static scan cannot resolve".
   - Its twin, the `cmd/testtiming/main.go:51` doc, was reworded to "a test above tier0 with a call the scan cannot resolve".
   - The review's F7 `fix` named only lines 151 and 168 of the pattern file, so this is a line the fix left as it was, but the two now disagree on wording.

3. **LOW**: `_lym/landing/summary.md` line 42 says the orch "kept the commands' `-tags tierN`".
   - That is consistent with the gate. It is worth confirming that the Verification section's change of command, from `--gate darn-write` to `--upto 2 ./...`, does not contradict the earlier line that the gate step names a `darn-write` gate. I did not read it, so treat this as unverified.

No inconsistency found in:
- the `tmuxAllowed` test against `tmuxallowed.go`;
- the battencli refusing seam message against the review's wording;
- the new `export` function name against its call site;
- the F7 testmain header rewordings.

**Tool calls**
- Two greps over the pattern, doc and crucible files for leftover `-tags` and untagged wording, to find twins of the changed text. My context had only the files the fixer edited.
- A grep of `internal/battencli`, `internal/testkit` and `crucible` for the refusing-seam message, `tmuxAllowed` and `tags tier`, to confirm the F6 and F2 names and to find the crucible twins.
