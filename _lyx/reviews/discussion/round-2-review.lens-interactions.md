```yaml
lens: interactions
outcome: reported
```

Restating my task: the interactions lens, i.e. the pairs of the record's decisions that act on the same state, and whether each pair still holds with both decisions in force.

**Pairs checked**
- D11 land × D9 create and D10 promote (a beam pair with no batten status file, then `lyx batten run` on it): broken (F1).
- D5 Add with `Parent` and the prime acting × D7 no dirt check for a pair forked from a beam: broken (F2).
- D11 first refusal (uses `ParentBeam`) × D9 members already spawned under a grouping project × D4 `ParentOf`: broken (F3).
- D10's AC "no batten start accepts after step 2 or later" × D1 and D11 (a beam project is startable as its landing): broken (F4).
- D10 start-guard refusal of a marked pair × D11 landing of a project: held, because the refusal covers tasks only.
- D3 owner setter with the beam flag × D10 step 5 kind change × D1 project field refusals: held.
- D2 run lock on `project`, `order` and `beam` × D9 step 1, D10 step 5 (claim already cleared at step 3) and D11 teardown settle (status, not scope): held.
- D2 refusal to change `project` on an entry with a local pair × D10 keeping the task's `project`: held.
- D2 rename refusal of a beam × D14 slug rule × D10 kind change making no slug: held.
- D1 no dependency on a project × D10 open-dependent refusal × prune of done dependents: held.
- D4 `ParentBeam` skipping done projects × D11 teardown marking the project done × D2 prune keeping a still-named done project: held.
- D2 prune exemption for `order` × D11 Land prune while the project is held by its landing: held.
- D4 origin-first `ParentOf` × D10 promoted beam whose origin parent differs from its `project` chain: held on the pair's machine; the bound is stated for other machines.
- D6 root pair with an empty parent × D4 empty parent reads as the prime × D12 unrecorded parent listed in the hub file × D8 guard naming the prime: held.
- D8 guard admitting a marked pair × D10 window between step 2 (mark written) and step 5 (board flag set): held, since D2 refuses `project` naming a non-project in that window.
- D12 mark written only by create and promote × D5 member weft forked from the beam's weft (the inherited `beam` key is a declared field, so `WriteJSON` does not keep it) × D7 adopt keeping the record: held.
- D9 steps 1→2 × D7 precondition way forward (branch absent → `lyx beam create`): held.
- D9 step 4 and D10 step 6 workspace regeneration × `regenerateAfterVerb` after each fabric verb: held, redundant but harmless.
- D8 `--window` in the parent's reed session × D11 landing window × pair teardown ending the beam's session × D11 live-orch refusal: held.
- D8 orch start refused under a landing × D11 live-orch refusal × D13 walk: held.
- D13 ready routing via `ParentBeam` × D8 orch filter via `ParentOf` (the same answer for an unspawned task): held.
- D5 bound that a member waits at Finalize on beam dirt × D7 × D8 operator working in the beam: held, and stated.
- D9 create on a hand-set `beam: true` × D3 bound: held.
- Constraint PATTERN-pair-teardown × D8 and D11, with the prime as working directory: held.
- The ACs of D3, D4, D7, D8, D9, D12 and D13 against their decisions: held.

**Findings**

F1 — BLOCKING — design — D11, with D9 and D10
- **Summary:** `lyx beam land` cannot start, because batten's start guard refuses its run.
- **Detail:** `lyx beam land` starts `lyx batten run <slug>` on a project whose pair already exists and which has no batten status file. D9 makes the pair outside batten, and D10 step 3 clears batten's records. Batten's start guard refuses a run or step with no status file when a task worktree, records worktree or records branch is present, naming `lyx batten cancel <slug> --remove`. That refusal fires before the Worktree-Create skip that D11 relies on ("Worktree-Create finds the pair present and skips"). D11 adds start-guard refusals for a project but never exempts a beam project from the left-pair refusal.
- **Evidence:** `internal/battencli/startguard.go:78-85`; `decision-record.md:145,163,182-185`.

F2 — MEDIUM — design — D5 with D7
- **Summary:** The prime's dirt still refuses a member forked from a beam.
- **Detail:** D5 and D7 pass the prime as Add's acting location for every parent the board picks. Add's opening check refuses `ErrSourceDirty` on the acting location's tracked dirt. So a beam member's start is refused by dirt in the prime, although D7 says no dirt check applies to a pair forked from a beam, and D5 exempts only the named parent's dirt. The record does not say which worktree's dirt Add judges when `Parent` is named.
- **Evidence:** `internal/fabricengine/add.go:188-193`; `decision-record.md:92,95,114,117`.

F3 — LOW — consistency — D11 refusal 1 with D9 and D4
- **Summary:** D11 waits on members that never land into the beam.
- **Detail:** The open-member refusal uses `ParentBeam`, so members that D9 says "keep the prime as their parent and land there" still block the beam's landing. D4 makes `ParentOf` the resolver that says where an entry's pair lands, and refusal 2 already covers local pairs by their origin record.
- **Evidence:** `decision-record.md:84,150,177-178`.

F4 — LOW — consistency — D10 acceptance criterion
- **Summary:** D10's acceptance criterion contradicts D1 and D11 after step 5.
- **Detail:** The criterion says a promote stopped "after step 2 or any later step leaves an entry no batten start accepts, naming `lyx beam promote`". After step 5 the entry is a project with `beam: true`, which D1 and D11 make startable as its landing run. D10's start-guard refusal covers only a task whose pair carries the mark.
- **Evidence:** `decision-record.md:44,167,274`.

Tool calls: none.
