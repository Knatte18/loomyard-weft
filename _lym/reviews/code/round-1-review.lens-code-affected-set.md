```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I checked what calls, reads, asserts or documents it.

**Per change, the set and whether it is covered**

- **`ArbiterEnvelope.Commits` and the verb's `commits` output key.**
  - Readers: `shedarbiter.Seam`, which maps it onto the outcome, and `arbiterEnvelopeFields`. Both are covered.
  - The cobra `Long` help is updated. Covered.
  - `docs/overview.md` mentions the fix. Covered.
  - `shed-template-driver.md:128` reads only `reason`, `question` and `verdict_path` on a hand-on. The question now carries the commits, so nothing in it is wrong. Covered.
  - The `internal/loomcli/doc.go` `# Arbiter` section (lines 441–494) describes the verb, `attempt.json` and the attempt directory. It is not updated. See F1.
- **`ArbiterAttempt.Snapshot` added to `attempt.json`.**
  - It is written in `settleFresh` and read in `readEdits`. Covered.
  - The JSON key is `omitempty`, and readers tolerate unknown keys (PATTERN-tolerant-records). Covered.
  - `loomcli/doc.go:465-469` describes `attempt.json` without the snapshot. Part of F1.
- **`ArbiterOutcome.Commits` and `driveAttemptNote.Commits`.**
  - They feed `arbitrate` and the stop report's attempt line. Covered.
  - `sheddriver/doc.go` is updated. Covered.
- **`SettleRecord.Commits`, the new `commits` key.**
  - `SettleRecord` is no longer comparable, so code that uses `==` on it would break. The `ParseSettleRecords` test was switched to `reflect.DeepEqual`. Covered.
  - `reportengine` uses `map[int]shedrun.SettleRecord`, which is a map value, not a key, so it is unaffected. Covered.
  - `drivewiring.go:189` compares `record.StopKey == key`, which compares the key field only. Covered.
  - The `shedrun` doc comment is updated. Covered.
- **`shedarbiter.Deps.Worktree` and `WorktreeSeam`.**
  - Loom wires it through `fabricengine.Open`, then `IsAncestor`, `CommitLog` and `worktreeSnapshot`. All of these are read-only. Covered.
  - Batten's `loadedArbiterDeps` leaves it nil. Covered.
  - PATTERN-fabric-boundary holds: fabric gains no new seam, and lym code runs no mutating git.
- **`PromptInput.MayEdit`, the `edit_scope` marker and the two fragments.**
  - Fragments are registered in `stencils.go` entries and pinned in `markers_test.go`. Covered.
  - The template's marker comment is updated. Covered.
  - Reads go through `stencilstore` (PATTERN-stencil-ownership). Covered.
- **Stencil wording in the arbiter template.**
  - In fix mode, "never edit a file / never run a mutating git command" became "Edit or commit anything the edit rules above do not let you" plus "Push, amend, rebase, reset or switch a branch". The no-fix fragment restores the old absolute rule.
  - The removed "a code change … never make yourself" is licensed by the Goal's idea.
  - "The verdict file is your only channel" (line 28) is still true, since commits are not a message channel.
  - None of the closed list's denied forms is spelled out (PATTERN-refusal-way-forward).
- **The new refusal "the worktree could not be read before the spawn".** It goes through `refusal()`, which adds `way forward:`, and a `TestCall_Edits` row reaches it. Covered.
- **Declared PATTERN changes:** the directive declares none, so there is nothing to check.
- **Commit messages:** the range has no `Pattern-Departure:` trailer, and nothing needs one.

**Findings**

- **F1 — LOW, consistency, `internal/loomcli/doc.go:441-494` (`# Arbiter`).** The package doc's Arbiter section is unchanged.
  - It describes the verb, `attempt.json` and the attempt directory's files.
  - It does not say that loom wires `Deps.Worktree` so its arbiter may commit one fix.
  - It does not say that `attempt.json` now records the worktree snapshot.
  - It does not say that the envelope and the verb's output list `commits`.
  - The sibling `doc.go` files in `shedarbiter` and `sheddriver`, the cobra `Long` help and `docs/overview.md` were all updated.
  - Evidence: `internal/loomcli/doc.go:443-472`, read in full above, against `internal/loomcli/arbiter.go:183-188,200` in the diff.

No other member of the affected set is left wrong.

Tool calls:
1. Grepped `shed-template-driver.md`, the docs, specs and skills for the changed keys and for arbiter edit/git wording. The manifest had not covered the agent driver stencil or the specs.
2. Read `shed-template-driver.md:110-140` and the arbiter template's channel lines. I needed the LLM driver's handling of the envelope keys, which the manifest lacked.
3. Grepped `driverStencilName` use and read the template's lines 20-40, to confirm the agent driver stencil is live and check the "only channel" wording against the new fix rules.
