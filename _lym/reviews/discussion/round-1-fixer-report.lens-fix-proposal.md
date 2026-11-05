```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as written (fix-proposal lens).

**1. MEDIUM: F15's change contradicts D14 and Open risks**
- **Changed text:** D13's new paragraph, "A scope record with no new check result but with the old compare, acceptance or waiver fields … renders those fields".
- **Case:** D13 now needs code that reads the old acceptance, waiver and compare fields of `scope.json`. Two places in the artifact say otherwise:
  - D14 removes "ruled and unruled acceptances and their halt marks" along with their tests.
  - The Open risks line still says the in-flight run "keeps acceptances in `scope.json` that no code reads".
- **Evidence:** `PATTERN-tolerant-records` only says unknown keys are tolerated, not that they are read. So the record model must keep these fields so the verdict can read them.
- **What to state:** D14 should say the record model keeps the old fields read-only for the verdict. The Open risks sentence that says no code reads them should be cut.

**2. MEDIUM: F4's darn route skips code review for the fix**
- **Changed text:** D7, "In darn that route is the parent's `lym loom goto --to Darn-Write` then `lym loom resume`".
- **Case:** At a Stuck Drift-Recheck, a goto to Darn-Write keeps the settled verdicts of the review segments that already ran. The help text says it "keeps the segment's settled verdicts" (`internal/loomcli/cli.go`, goto Long), and `docs/overview.md:552` says re-entering Darn-Write "keeps the spent review budget and any settled review verdict". So code the writer changes to fix the regression can reach Drift-Recheck and landing with no new Code-Burler review.
- **What to state:** D7 should say whether the review rows run again after that goto, or accept and state this bound.

**3. MEDIUM: F20's refusal fires on ordinary output, and its way forward breaks the sealed expectation**
- **Changed text:** D1, "A key that is empty or ends in `#` followed by digits … the diff and Drift-Expect's identity listing refuse it, naming … `drift.key_patterns` as the way forward".
- **Case:** The default `name: value` pattern keys a line such as `Test #3: passed` as `Test #3`, so a common harness line breaks every drift row.
- **Why the way forward fails:** If the refusal fires at Drift-Verify, editing `key_patterns` mid-run changes how every keyed and unkeyed line is identified after the expectation was sealed against the old listing. The sealed entries then cover nothing, every cell is unexpected, and the run ends in a Stuck needing `amend`.
- **Alternative:** The review offered one: an occurrence separator that a key pattern cannot capture. It avoids the refusal entirely.

**4. LOW: F10's "moved HEAD" doesn't say which HEAD**
- **Changed text:** D9, "snapshots HEAD, the tracked tree and the untracked paths … a moved HEAD … or a new untracked path other than `judgement.yaml`".
- **Evidence:** `_lym` is a symlink into the weft checkout; `git check-ignore` on `_lym/...` fails with "pathspec is beyond a symbolic link". `judgement.yaml` lives under `_lym/drift/`, on the weft.
- **Case:**
  - If the snapshot is taken only in the warp, a judge that commits or writes files on the weft is not caught.
  - If it is taken in both, any other note the session writes under `_lym`, such as a friction note, becomes a Stuck.
- **What to state:** D9 should name the warp and weft HEADs, and any `_lym` paths the session may legitimately write.

**5. LOW: F16 combined with F6's bound lets an empty ruling skip Drift-Recheck**
- **Changed text:** D4, "in `command` mode the compare exited 0 or the judge ruled every case it read `matches`".
- **Case:** A judge that lists no case meets "every case it read" vacuously. D2's new Bound already says such a judge passes. With F16, that run also counts as a passing check, so Drift-Recheck at the same head is Done at once with no second look.
- **What to state:** D4 should require at least one ruling when the compare failed, or state this explicitly.

Tool calls:
- `grep` of `internal/shedengine/status.go` plus `.gitignore` and `git check-ignore`: to learn whether run status stores the recipe and which drift paths are ignored or symlinked. My context lacked both. The `check-ignore` call exposed the `_lym` symlink.
- `grep` for "recipe" in `status.go` and for retired-row files: to confirm that a run uses the current binary's recipe, so F11's row-order test holds for in-flight runs. It does; no finding there.
