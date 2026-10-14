---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (reversal-failure correctness-halt row)
    summary: The reversal-failure halt names a hand reversal by the orch as its way forward, which is none of the three forms refusal-spec.md admits, and its envelope of old and new file names cannot restore a deleted card or a rewritten heading or Commit prefix
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (raw I/O reversal paragraph)
    summary: The reversal is built from the in-memory move list of new names and contents, but undoing a deletion, a heading or Commit-prefix rewrite and the Card Index edit needs every touched file's pre-call bytes, and D2 never says the verb holds them or where the create or delete step sits in the apply order
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2; contracts/specs/refusal-spec.md loom raw I/O row
    summary: D2 says a mid-apply I/O failure joins the loom section's grouped raw I/O row, but that row covers stat, mkdir, read, write or remove of status, seed, lock, park marker or records files, not a rename or a plan card or overview file, and D2 does not widen it
---

### [BLOCKING] Reversal-failure halt has no admissible way forward

**Location:** `_lyx/discussion/decision-record.md`, D2, the paragraph starting "A failure of that reversal is its own correctness-halt row".

**Issue:**
`contracts/specs/refusal-spec.md`'s "What a way forward is" admits three forms: a `lyx` verb, a transition the run takes by itself, or a plain `git` command in the task worktree.
D2's way forward is "the refused session reports status FAILED naming the envelope and the orch reverses the listed renames and deletions".
That is none of the three: it is a manual file operation by another session.
The spec's `status: FAILED and the orch runs it` clause exists only for a `lyx fabric` verb the role guard refuses from a task session, so borrowing it here does not make a hand reversal conform.
This breaks `PATTERN-refusal-way-forward` ("names its way forward in its message, as that file defines one"), which the decision record lists under Constraints.

The step is not executable even as written.
The envelope lists each unreversed step "as old and new file name".
A moved card also had its `# Card M` heading and explicit `**Commit:**` prefix rewritten, which a rename does not undo.
And a deleted card cannot be restored from its file name: its bytes exist only in the dead process's memory, and the plan directory may hold uncommitted edits from the current Plan-Write or Plan-Burler session, so `git checkout HEAD -- <path>` brings back a stale version.

**Fix:**
Give the row a way forward of an admitted form that restores the pre-call bytes.
For example, the verb writes every file it is about to change or delete, with its pre-call bytes, to a backup under the mirrored `.lyx` path before the first apply step, and the halt names a `lyx loom` restore verb or re-run that puts them back.
The other option is to refuse a plan directory with changes against HEAD before applying, so that `git checkout HEAD -- <plan dir>` and a plain `rm` of the created file restore the directory losslessly, and name that command pair.
Either way, state in D2 what the envelope or the backup carries, and add the restore path to the acceptance criterion for an injected reversal failure.

### [MEDIUM] Reversal source and apply order underspecified

**Location:** `_lyx/discussion/decision-record.md`, D2, the paragraphs "The verbs compute every new file name and content in memory first" and "A raw I/O failure mid-apply reverses the steps already applied".

**Issue:**
D2 says the verbs compute every *new* name and content in memory, and that the reversal runs "from the in-memory move list".
Undoing an applied step needs the *old* state: the old name and old bytes of each moved card, whose heading and `Commit:` prefix changed; the full bytes of `remove-card`'s deleted file; and the overview's pre-call Card Index.
D2 never says the verb keeps those bytes, so the acceptance criterion "A raw I/O failure injected mid-apply leaves the plan directory byte-identical" rests on an unstated mechanism.
D2 also orders only the moves and the Card Index; it does not place `insert-card`'s new-file write or `remove-card`'s deletion.
Under "no two files ever claim one number", the deletion must precede the first downward move and the creation must follow the last upward move, and the reversal of a deletion must come last.

**Fix:**
State in D2 that the verb reads and holds the pre-call bytes of every file it will rewrite or delete before the first apply step, and that the reversal restores from those bytes.
State where the create and delete steps sit: `remove-card` deletes card `N` before the ascending moves, `insert-card` writes the new card after the descending moves, both before the Card Index.

### [MEDIUM] Mid-apply I/O failure joins a raw I/O row that does not cover it

**Location:** `_lyx/discussion/decision-record.md`, D2; `contracts/specs/refusal-spec.md` loom section, the `raw I/O` row.

**Issue:**
D2 says the reversed failure "joins the loom section's grouped transient raw-I/O row, whose 'nothing is mutated' then holds".
That row reads "`stat`, `mkdir`, `read`, `write` or `remove` of a status, seed, lock, park marker or records file fails".
A failed `rename`, or any I/O on a plan card file or `00-overview.md`, is outside it, so the failure has no row as the spec stands.
Scope's "Their refusals, as rows in `refusal-spec.md`'s loom section" names new rows, not a widened existing one, and the acceptance criterion "Each new refusal has its row" would not catch the gap.

**Fix:**
State in D2 that the loom section's raw I/O row is widened to name `rename` and the plan directory's card and overview files, and add that row edit to Scope or to the refusal acceptance criterion.

## Focus verdicts

- **Failure path and refusal ways forward.**
  Reached: a reversed mid-apply failure leaves the plan as it was, so the precondition refusal never sees a half-renamed plan on re-run, provided the reversal works (F2) and the raw I/O row covers it (F3).
  The unreversed case's way forward is not admissible (F1).
- **Slug and intent bounds.**
  Not a defect.
  D2 defines the slug rule itself and says so, which excludes `/` and `..` from the name `cardFileName` builds.
  The intent bounds match `cardIndexLineRe`: a whitespace-only intent trims to a line the `(.+)$` group cannot match after the separator's `\s+`, and a line break yields a second, unparseable index line.
  An intent containing an em dash still parses, since `\S+` takes the slug before the second separator.
- **Begun-card bound atomicity.**
  Not a defect.
  D2 holds `AcquireStateMutation` from the run-record read through the last apply step, and states the begun set as every batch record, matching `DispatchScope`'s `begun` rather than `DoneCards`.
  `webstercli`'s begin-batch parses the plan before taking the lease, but `BeginBatch` calls `PlanEditError`, which fingerprints the plan from disk under the lease, so a begin-batch that waited out a renumber refuses with a fingerprint mismatch instead of recording a batch against stale cards.
- **Open risks and D5 scope.**
  Not a defect.
  Open risks is `None.`; the rebaseline question is settled under Constraints, and the claim matches `autoRebaseline` (names every changed card file's number) and `changedPlanFiles` (counts a vanished file as changed).
  D5's writer rule and rubric bullet are both scoped to a refusal or guard branch a card prescribes, with the grouped raw I/O exemption.
