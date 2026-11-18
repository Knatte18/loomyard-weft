```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record's fit with the board entry `hardener`, its peers and the touching notes.

**Findings**

1. **MEDIUM, class `decision`. D8 and Open risks: retiring Treadle goes against the operator's own wording in the entry.**
   - The entry's "Operator decision (2026-11-18)" says to design Hardener "on lym's own modules as they stand today (`shed`, `treadle`, `burler`, the gate and the sandbox work)".
   - The entry's last Status bullet records an operator decision of 2026-10-16: the treadle stencils "are kept for Hardener ... and are re-derived from the Bouncer judge when Hardener is built".
   - D8 deletes `internal/treadleengine`, its stencils and its roles. The Open risks line settles this itself, by saying the operator note "allows" it because the draft is "only an account".
   - That note calls only `crucible/README.md` and the draft an account. Treadle appears in the operator's list of modules to build on.
   - Question for the parent: may Hardener be built on the Bouncer with treadle deleted, or must treadle stay, as the round loop or kept unused?

2. **LOW, class `decision`. Scope Out: "the sandbox work" may name the board note `sandbox-closed-area-build`, which the record rules out.**
   - The operator's list names "the sandbox work" as something to build on.
   - The record excludes the bwrap closed area of `sandbox-closed-area-build`, an unbuilt note, and builds its own throwaway local hubs (D5).
   - If "the sandbox work" means `tools/sandbox` and the dev-bin deploy, there is no conflict. If it means F9, D5 replaces the work the operator pointed at.
   - Question for the parent: which sandbox work did the operator mean?

3. **LOW, class `consistency`. The entry's Notes point is not addressed.**
   - The point reads: "One of the Someday items on `manifest-retire`'s triage list (`hardener`). Landing this entry removes it from that list."
   - The record never mentions the triage list. The writer's grep for `manifest-retire` found nothing outside `.lym`, so the list may no longer exist in the repo.
   - Fix: say in the record (Scope or Notes for the plan writer) that the list is absent, so there is nothing to remove, or name where it lives.

**Goal points of the entry**
- `lym` starts a Hardener campaign against a module and runs it unattended to convergence or a round cap: met (D1, D2, D6 and the cap in D9).
- `crucible/` is deleted and its durable content lives in package docs: met (D12).
- The four "Open before Discussion" questions are all answered:
  - What the handoff carries: D6.
  - Where the line falls between Go and the LLM: D4 to D7, ledger Q3.
  - Whether the round agent reuses the burler package: D3.
  - How much sandbox provisioning lym automates: D5.
- Build on lym modules as they stand, not by porting crucible: met for shed, burler and the gate. Open for treadle and "the sandbox work" (findings 1 and 2).
- Removal from the `manifest-retire` list: not addressed (finding 3).

**Entries, peers and parent commits checked**
- `testmap-reach-triage-mutate` (T7, peer): both touch `internal/shedrecipe`, `internal/shedadapters`, `internal/gateslot` and the verify gate. That is a text-only package overlap, so no conflict.
- `fabric-single-repo-mode` (T1, peer): `lym board get` returned `task: null`, though `lym board list` and `lym loom peers` showed it running. I could not reach it. It may bear on where D7's `_lym/harden/` archive lives, but I have no evidence of a conflict.
- `drop-unused-features` (E7 note, code-cleanup project): no conflict. It notes that `remove-unreachable-code` has landed, and D8's treadle removal fits that project's direction.
- `sandbox-closed-area-build` (F9 note): no conflict apart from finding 2.
- Parent commits: `lym loom peers` lists none.

**Decision notes**
- `lym board list` shows no `Decision for hardener:` note, so I read none.
- The record carries no `Added after Discussion` entries.

**Tool calls**
- `lym board get fabric-single-repo-mode`: my context held only its list line, not its body, and the call returned null.
- `lym board get testmap-reach-triage-mutate`: my context had only its title and changed paths, not the packages it changes.
- `lym board get drop-unused-features`: my context held only its brief, not whether it already lists treadle.
