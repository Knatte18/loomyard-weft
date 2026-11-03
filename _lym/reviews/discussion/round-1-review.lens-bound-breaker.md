```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked — held / broken**

- D1 "file is no render output, so a render never deletes it": held for default prefix; **broken** — refusing only a `design_prefix` *equal to* `decision-` still lets the render write over a record file (F1).
- D1 "rides the board's own Bolt commit": held — `commitDirty` → `bolt.Commit` stages the working tree (internal/boardengine/sync.go:65-75).
- D2 "`scopeChanged` never compares `discussed`, so a run-held entry takes the write, no revised date": held (runlock.go:189-213, 138-164; dates.go:48).
- D2 "file before save, so a crash never leaves a flag without a file": held for a crash in the store; **broken** by an older binary's rename (F2).
- D2 "the run ledger sets aside a board commit whose entries have all ended": held (fabricengine/replay.go:635, 693; boardengine/replay.go:252 `Ended: isDone`).
- D2 "only Go callers reach the method": **broken** (F4).
- D2 refuse done entry vs D8/D9's way forward: **broken** (F3).
- D3 "dropped as a no-op, as the `beam` and `front` keys are": held (store.go:507-512).
- D4 "a rename moves the record file in the same write": **broken** unless the move runs after the save (F5).
- D4 status-only cancel keeps flag and file: held.
- D5 "replays by the design-file rule": held for the mechanism (boardengine/replay.go:532-566).
- D6 "seam before commit seam, after carry-over, on every approving path": held — `approveAndCommit` after `writeCarryOver` in `settle`, `acceptRound`, `approveWithoutReview` (bouncer.go:509-544, 1164-1168, 1273-1279).
- D6 "a goto past Discussion-Bouncer skips the store": held.
- D8 bound (mid-settle call): held as stated.
- D9 "it can flag an entry discussed only where the run's approved record exists": **broken** by the D9 bound's own next clause (F6).
- D9 "each judged as `decision add` judges that caller": weak (F7).
- Auto-mode #3 "the orphan is harmless": held only for the orphan file; the flag side breaks (F2).
- Support-log rejected finding "flag with no file reachable only by hand deletion": **broken** (F2).

**Findings**

F1 — MEDIUM — design — D1 (`design_prefix` refusal)
Refusing only a prefix equal to `decision-` does not keep the render off record files. `RenderToDisk` writes every rendered file with `AtomicWrite` before removing stale manifest entries (render.go:48-52), and `renderDesigns` names files `<design_prefix><slug>.md` (render.go:795). Counterexample: `design_prefix: dec` and an entry slugged `ision-foo` renders `decision-foo.md`, overwriting the record of entry `foo` with design-doc text; a later render then deletes it via the manifest. Any prefix that is a proper prefix of `decision-`, or starts with `decision-`, collides the same way. Fix: refuse every `design_prefix` for which `<prefix><slug>` can match `decision-<slug>.md`.
Evidence: internal/boardengine/render.go:43-63, 772-797; config.go:31 (no validation exists today).

F2 — MEDIUM — design — D2 invariant vs Auto-mode assumption 3
"Never a flag without a file" breaks under an older binary's rename: the constraint states an older binary keeps `discussed` as an unknown key and writes it back, so an old binary's rename of discussed entry `a` → `b` carries `discussed: true` onto `b` while the file stays `decision-a.md`. Result: `b` flagged with no file, `decision-a.md` an orphan. Assumption 3 covers only the orphan file; the support log's rejection of the D10 finding claims this state needs hand deletion, which is false. D3's rationale names exactly this state as what breaks the follow-up's gate.
Evidence: decision-record.md lines 129, 140; support-log.md line 35; rename.go:26-66.

F3 — LOW — consistency — D8 and D9 vs D2 (done entry)
D8 sends a failed copy to `lym loom decision store`, but on a done entry D2 refuses and the same verb would refuse identically. A run whose entry Finalize has already marked done (worktree still present before teardown) meets a way forward that is itself refused. Neither D8 nor D9 states what a post-landing call meets.
Evidence: decision-record.md lines 43, 104, 111-115.

F4 — LOW — consistency — D2 bound vs D9
"Only Go callers reach the method, since no board verb writes either" understates reach: D9 adds a CLI verb callable by the run's parent orch, the operator's shell, and the run's arbiter (an LLM strand). The parent orch is the party the run lock exists to restrain. D2's bound should name D9 as the one hand-reachable route and refer to D9's own bound.
Evidence: decision-record.md lines 48, 112.

F5 — MEDIUM — design — D4 (rename move timing)
D4 says "in the same write" without saying where. If the move happens inside the mutation (as D2's own file-first precedent does), a rename refused by the post-mutation checks still leaves the file moved. Counterexample: `checkNewSlugs` (pair-slug rule run with `renamedTo` after `fn`) refuses the target; board.json keeps the old slug flagged while its file now sits at `decision-<new>.md` — a flag without a file plus an orphan. rename.go today guarantees "Every refusal happens before the save, so a refused rename writes nothing", which a move inside the mutation breaks. Fix: the rename's move runs after the save (in `afterSave`, like D4's removals).
Evidence: internal/boardengine/rename.go:3; board.go:103-149; slugrule.go:17-32.

F6 — LOW — consistency — D9 bound
"It can flag an entry discussed only where the run's approved record exists" is contradicted by the same bound's next clause: after a goto past Discussion-Bouncer the record was never approved. D8 also copies in that state and says nothing about it.
Evidence: decision-record.md lines 101, 115.

F7 — LOW — consistency — D9 caller rule
`decision add` admits the arbiter only with `--by arbiter` plus an arbiter-strand check; D9 has no `--by`, so "judged as `decision add` judges that caller" leaves the arbiter test undefined. State that D9 uses the arbiter-strand check alone.
Evidence: internal/loomcli/decision.go:8-9, 645-648.

Tool calls:
- render.go 40-66 (stale-manifest removal loop) — my context only had grep lines, not the write-then-remove order.
- store.go 495-530 + RunHolds — to confirm the beam/front payload no-op and the run-hold predicate behind D3/D2 claims.
- rename.go refusals + checkNewSlugs — to see whether a rename can be refused after the mutation runs, which F5 rests on.
