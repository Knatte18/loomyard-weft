```yaml
lens: bound-breaker
outcome: reported
```

## bound-breaker lens report: umbrella-groundwork decision record

Of the 21 claims I attacked, 11 held and 10 broke (6 MEDIUM, 4 LOW).

**Findings**

1. **MEDIUM** · D13 · A stopped beam orch keeps receiving notices and escalations, so the walk to the prime never happens.
   - D13 walks to "the nearest worktree whose orch state records a strand".
   - `orchcli/stop.go` removes the strand (`RemoveStrand`), but nothing clears `State.Strand`. It is set only at `orchengine/state.go:445`, by start.
   - Counterexample: a beam orch is started once and then stopped with `lyx orch stop`. Every later notice resolves to the dead `ly:<beam>:orch`.
   - The record needs a liveness predicate or a cleared-on-stop rule.

2. **MEDIUM** · D1 · A machine or long-lived process still on an older binary breaks on `kind: project`.
   - In an older binary, `normalizeTask` refuses any kind other than task or note. It runs in the board replay (`replay.go:283` and `replay.go:365`), so that machine's replay of a rejected push fails.
   - In the same binary, `layer.go:196` and `render.go:303` treat every non-note as a task. An older orch or README render would show a project as a startable task.
   - The record states no rollout bound: deploy order, other machines, or an old batten window still running.

3. **MEDIUM** · D2 vs D11 · The done-project refusal makes the landing's own write, or any later write, refuse.
   - D2 refuses a write that "leaves" an entry naming a done project.
   - D11's teardown marks the project done while its done members still name it, so that write leaves exactly that state. The same happens when an operator sets a grouping project to done by hand.
   - If the check runs over all entries, every later board write refuses until prune. D2 has to scope the check to entries whose `project` the write changes.

4. **MEDIUM** · D10 / D1 · Promote's kind change meets the existing "only tasks may be depended on" rule.
   - `store.go:367` refuses making an entry a note while tasks depend on it, and D1 lets a project carry no `depends_on` relation at all.
   - Counterexample: task B has `depends_on: [A]`, and `lyx beam promote A` flips A to a project. Either the write is refused or B dangles.
   - D10 does not say which, or what happens to the dependents.

5. **MEDIUM** · D9 / D10 (create on a grouping project) · Members started during a create, or already running, land in the prime, not the beam.
   - D9 writes `beam: true` last, at step 3. A member started between steps 1 and 3, by the orch or later dobby, has `ParentBeam` read the flag as false and forks from the prime.
   - Members already spawned under the grouping project also keep the prime as parent.
   - D10's "its tasks need no change" holds only for unspawned members. The record is silent on in-flight ones; the land check passes once they are done, but their work never reached the beam.

6. **MEDIUM** · D6 bound · "Only an operator at the terminal adopts a local branch" is false.
   - The orch run loop in `orchcli/doc.go`, step 1, has the LLM orch run `lyx fabric add <slug>`.
   - Under D6 that orch would silently adopt a stale leftover branch under a reused slug, which is exactly the case the bound claims to exclude.

7. **LOW** · D1 · Upgrading a hub with a type label starting with `P` breaks the board.
   - `LoadConfig` refusing such a label (`proposal`, `performance`) fails every board read and write on any hub with one. This hub has only A, B, E and F; other hubs, such as the Models hub, are unchecked.
   - The record states no migration or refusal path.

8. **LOW** · D4 · `ParentSlug` cannot signal "no pair".
   - `fabricengine.ParentSlug` returns `("", nil)` both when no origin record is found and when the parent is the prime (`origin.go:139-153`).
   - "Falls back to `ParentBeam` for an entry with no pair" needs a found signal it does not have. A spawned task of the prime would otherwise fall through to `ParentBeam`. That is harmless only while the board and the origin agree.

9. **LOW** · D11 · The live-child-pair check sees only this machine's pairs.
   - Counterexample: a pair created by hand with `lyx fabric add x --parent <beam>` on another machine, with no board member. The member check passes, the local scan finds nothing, and the beam lands and is torn down under that pair.

10. **LOW** · D14 · No behaviour is stated for a board write where fabric's junction names cannot be loaded.
   - The slug seam needs fabric config, but board writes also run where no worktree resolves. The detached `lyx board sync` runs in the board dir, and a Board built from a bare path validates nothing.
   - Whether the check is skipped or refuses there is not stated.

**Claims attacked**

| Claim | Result |
|---|---|
| D6: the Land row stops on an origin record with no parent branch | held (`land.go` ~190) |
| D6: only the operator adopts | broken (#6) |
| D13: "records a strand" predicate | broken (#1) |
| D1: new kind is safe under unknown-key tolerance | broken (#2) |
| D1: `P` letter refusal | broken (#7) |
| D2 / D11: done-project refusal | broken (#3) |
| D10 / D1: promote kind change | broken (#4) |
| D9: board flag last, so no member forks from a missing beam | held for missing pairs, broken for in-flight members (#5) |
| D4: fallback via `ParentOf` | broken (#8) |
| D11: start guard blocks a resume | held: D2's run-status refusal closes it |
| D11: child-pair check | broken (#9) |
| D14: slug seam | broken (#10) |
| D5: `Parent` empty keeps today's fork from the prime | held |
| D8: bookend refusal of a run from inside the pair | held |
| D8 / D11: landing window in the parent's reed session survives the beam's teardown | held |
| D12: unmarked file kept, marked file removed | held |
| D12: pairs with an unrecorded parent go to the hub file | held |
| D7: dirty check on the beam worktree | held (consistent with today's prime check) |
| D3: hand-edit bound stated | held |
| D10: refusal past Finalize's parent-side merge | held: cancel's `ChildReachedParent` seam already judges it |
| D8: beam orch rules left unchanged at landing | held as a requirement; no mechanism stated, plan's to find |

**Tool calls**

1. Grepped `orchcli/stop.go`, `orchengine/state.go`, `normalizeTask` call sites and `ParentSlug`: the context lacked whether stop clears the strand, where kind validation runs, and `ParentSlug`'s return semantics.
2. Grepped the board's kind handling in `store.go`, `migrate.go`, `layer.go` and `render.go`: the context lacked how existing code treats a kind other than task or note.
3. Grepped orch stop and state for strand clearing: confirms nothing clears `State.Strand` on stop.
