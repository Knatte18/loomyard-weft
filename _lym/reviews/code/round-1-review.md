---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/heddleengine/store.go:271
    summary: heddle set refusals name a --target flag the CLI does not have; the flag is --targets
    fix: In internal/heddleengine/store.go change the "--target" strings in missingFlags (line 271) and in the ValidateEntry no-target message (line 297) to "--targets", matching the flag declared at internal/heddlecli/verbs.go:146. Also make `heddle set <slug> --targets ""` on an existing entry refuse naming --targets instead of leaving the targets unchanged, for example by passing a non-nil empty slice from setCmd when the flag was given, so ValidateEntry rejects it.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/heddleengine/store.go:162
    summary: Decode's way-forward text names a placeholder Bolt path and advises restoring HEAD, which is wrong for a replay blob that failed to decode
    fix: In internal/heddleengine/store.go make Decode (used by replayer.go:63-73 for base, parent and commit blobs) return a refusal that says which side failed and that the commit holds a malformed heddle.json, with a way forward that fits a git blob (fix the file in a new commit on the local branch), and keep the `git checkout HEAD -- heddle.json` advice only on the Load path through decodeFrom, which has a real boltDir.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: docs/overview.md:172
    summary: docs/overview.md still describes fabric.yaml as read via fabricengine.BoardDir and spells the _board literal, which the Bolt checkout now owns
    fix: In docs/overview.md line 172 name the Bolt checkout and fabricengine.BoltDir instead of `<BoardDir>` and fabricengine.BoardDir, and reword lines 119, 126, 143, 182 and 370 to say "the Bolt checkout" in place of the `_board` literal where they describe the checkout rather than the board's own files.
    origin: lens:code-affected-set
---

### [LOW] heddle set refusals name a flag that does not exist

**Location:** internal/heddleengine/store.go:271, 297
**Issue:** `missingFlags` and the no-target refusal say `--target`, but the verb declares `--targets` (internal/heddlecli/verbs.go:146). D2 requires every refusal to name a way forward that works. A `--targets ""` on an existing entry also yields a nil patch and silently changes nothing.

### [LOW] Decode's way forward is wrong for a replayed blob

**Location:** internal/heddleengine/store.go:162
**Issue:** `Decode` passes the placeholder `<Bolt checkout>` to the refusal, which tells the operator to restore HEAD's file. Replayer.Apply decodes the merge base, parent and commit blobs with it, so a malformed historical blob gets advice that does not repair it.

### [LOW] docs/overview.md is stale after the Bolt directory name change

**Location:** docs/overview.md:172
**Issue:** The text still names `fabricengine.BoardDir` as where fabric.yaml is read from and spells `_board` throughout. Fabric's config now comes from `BoltDir`.

## Rejected

- Lens code-tests F1 (no resume test for the append-system-prompt file): the diff adds no resume code, and `TestPrepare_AppendSystemPromptMatchesDenyNotice` already pins that both the launch and resume lines carry `--append-system-prompt-file`.
- Lens code-tests F2 (no zero-passage case): the join is a loop that does nothing for an empty list, and the existing no-reader row reaches the same output.
- Lens code-tests F3 (claude and codex prompt separation): `ShowAndPassagesWriteNothing` asserts `Passages(claude)` returns its entries and `Passages(codex)` returns none.
- Lens code-correctness F2 (`FastForwardPair` fires take-in): it moves the pair to the fetched origin upstream, which is a take-in from origin, and plan card 10 names it.
- Lens code-correctness F3 (30-second Bolt lock wait on a launch): D3 and D4 specify the bounded wait and the fall back to lym's sentences, so the behaviour is as decided.
- Lens code-affected-set F2 (`_board` in tools/sandbox suites, README): outside D9's list of Go code, tests, stencils, specs and skills.
- Lens code-affected-set F3 (no docs for the new surfaces): no repo rule found that names a doc for this kind of change.
