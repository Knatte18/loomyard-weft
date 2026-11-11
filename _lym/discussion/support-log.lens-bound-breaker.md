```yaml
lens: bound-breaker
outcome: reported
```

The record's main weaknesses are in D2 and D5. D2's trailer cannot be carried by any lym commit path that exists today, and a sweep commit can stop it from touching the record at all. D5's warning fires on nearly every Sketch.

**Findings**

1. **BLOCKING — D2 / Constraints: no lym commit path can carry the `Lym-Record-Replaced:` trailer, and the Constraints line contradicts D2.**
   - The Constraints say the replace commit "goes through the fabric commit path `add` uses". The way forward on `add`'s commit failure names `lym fabric commit`.
   - `lym fabric commit --help` says its message is always the fixed string "weft sync", with only the `Warp-SHA:` trailer, and "cannot be customized with a flag".
   - So D2 needs a new fabric commit seam that takes a caller trailer. The record says the reverse.
   - Fix: D2 states that fabricengine gains a commit with a caller-named trailer, and the Constraints line stops claiming the existing path.

2. **BLOCKING — D2 bound: a sweep commit can land the replaced bytes first, so the trailer commit never touches the record file and the reset never happens.**
   - Weft commits stage all of `_lym` ("weft sync" sweeps `_lym` wholesale).
   - `add` already defends against "a status commit landed the amendment record" (`rollBackAppend` and `afterFailedCommit` in `internal/loomcli/decision.go`). That shows another commit can sweep the verb's working-copy write before its own commit.
   - Counterexample: replace writes a record that drops an Added entry. A status commit or `lym fabric sync` commits it as "weft sync". The replace's own commit then has no change to the record, or has nothing to commit at all.
   - `CommittedAnchoredFileHistory` lists only commits that touched the path (`repo.PathRevisions(scoped, 0)`), so no trailer commit appears for the file.
   - The history then holds the pre-replace versions. Every later gate reports `added-decision-dropped`, and the run cannot pass.
   - The same happens when the replace's commit fails and the named way forward, `lym fabric commit`, lands the record without the trailer.
   - Fix: the reset must not depend on which commit first carries the bytes. Options:
     - Record the replace in a Go-owned run-side marker file under `.lym` or the run dir, holding the replaced record's hash. The history check then truncates at the newest version whose bytes match that hash.
     - Or have the trailer search fall back to the newest version byte-equal to the replaced record.

3. **BLOCKING — D5: the warning fires on nearly every Sketch a motif run makes.**
   - Discussion approval stores the record, which stamps `record_stored`.
   - Later in the same run, `sketchFinish` → `MakeSketch` turns the task into a note. It changes `Kind` and drops `Recipe`, `DependsOn` and `Isolated`.
   - All of those are compared in `scopeChanged` (`internal/boardengine/runlock.go`), so `stampDates` sets `Revised = now`, which is later than `record_stored`.
   - `stampDates`' promote exemption covers only `Kind` and `Priority`, and only at promote time.
   - So D5's "equal times → no warning" criterion never holds on the main path, and the warning becomes noise.
   - Fix: also stamp `record_stored` in the `MakeSketch` write (and in Raddle-File's filing write). Or compare only `Body` changes: give the body its own `body_revised` time instead of using `revised`.

4. **MEDIUM — D1 base-hash guard: the check and the write are not atomic.**
   - My grep found no lock around the decision verbs in `internal/loomcli/decision.go`.
   - Counterexample: replace A checks the base and passes. Arbiter `add` B appends and commits. A writes its file and commits.
   - B's entry is now silently dropped. D2 then exempts the drop, because A's commit is the newest replace.
   - Two concurrent replaces with the same base likewise both pass.
   - Fix: D1 states that `add`, `replace` and `store` serialize under one per-run decision lock, held from the base check through the commit.

5. **MEDIUM — D4 and D3 interleaving: a board replace can be silently reverted.**
   - The run lock holds an entry only once its status reads `<state> · <producer>`.
   - Counterexample: a Sketch is promoted to raddle. Record-Write seeds the old record. In the window before the status shows a producer, or in any gap the lock does not cover, the operator runs `upsert --record-file` and it passes.
   - The parent's next `add` on the raddle run then copies the run's old record plus the entry back onto the board through D3. The operator's newer record is gone, with no signal.
   - Fix: D4 refuses whenever a live run or worktree exists for the slug (batten's run record), not only on a held status. Or D3 refuses to overwrite a board record whose `record_stored` is newer than the run's seed.

6. **LOW — D5: a payload can set `revised` itself.**
   - `stampDates` stamps only when `after[i].Revised.Equal(before.Revised)`, so an upsert that names `revised` can skew the comparison either way.
   - D5's bound names only title and label changes. It should also say the payload can set `revised`.

7. **LOW — D4 with D5 in one write: two clock reads could produce a spurious warning.**
   - If `record_stored` and `revised` are stamped from different clock reads, `revised` can be nanoseconds later.
   - D4 should say both are stamped from the one `now` that `stampDates` takes.

8. **LOW — D5: board replay may not carry the new field.**
   - Board replay (`internal/boardengine/replay.go`) copies `Revised` from the commit explicitly.
   - The record should say whether `record_stored` is carried the same way. If it is not, a replayed entry has `revised` but an old or zero `record_stored`.

**Claims attacked**
- D1 caller guards reuse `add`'s: held.
- D1 base hash prevents a silent drop: broken (finding 4).
- D1 refusal before approval: held. A discussed tapestry run that skips Discussion is past its approval row.
- D1 carry-over blocks kept byte for byte: held.
- D1 identity change → re-review on re-entry: held, and it is stated.
- D2 trailer written by the verb: broken (finding 1).
- D2 reset reaches the history: broken (finding 2).
- D2 only Go writes the trailer: held. `lym fabric commit`'s message is fixed, so neither an agent nor the orch can forge it.
- D2 every history caller is bounded: held, unverified beyond the shared `recordHistory` seam.
- D3 refreshes stored records: held, but it interacts with D4 (finding 5).
- D3 copy failure is a non-refusal: held.
- D4 refused while a run holds the entry: broken at the edges (finding 5).
- D4 one board write, file before `board.json`: held.
- D5 warning only when the body is newer: broken (finding 3), weakened (findings 6 and 7).
- D5 entries with no `record_stored` give no warning: held.
- Open risk on notice-only reach to a live agent: held.

**Tool calls**
- Grepped `dates.go` and `sketch.go` for `scopeChanged` and `MakeSketch`: my context lacked which fields stamp `Revised`.
- Read `decisionWayFabricCommit`, decision-verb locking and `CommittedAnchoredFileHistory`: my context lacked the commit path, the locking and the history semantics.
- Grepped for `Revised` stamping sites and the `stampDates` body: my context lacked the promote exemption and the explicit-`revised` rule.
- Read the `scopeChanged` body in `runlock.go`: my context lacked the scope field list.
- Ran `lym fabric commit --help` and grepped decision.go for locks: my context lacked whether the commit message can be set and whether the verb takes a lock.
