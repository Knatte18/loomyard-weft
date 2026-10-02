---
verdict: APPROVED
rationale: "The decision record already carries every round-1 fix (D1 bound covers shape mismatch, list elements and extra keys; D2 names the --if-absent liveness set; D4 orders PathTracked before writes; D6 is bound to warp worktrees; Goal names the one-time spawn; D8 has a criterion), and every new flag, overwrite and downgraded guard states its bound."
---

# Bouncer verdict — discussion, round 1

The round-1 report is `APPROVED` with eight findings (two MEDIUM, six LOW) and no `## Focus departures` section, so there is no departure to ratify or reject.
I checked each finding's fix against the current `decision-record.md`; each is present:

- F1 (shape mismatch): D1's Bound now fills only where every ancestor the file holds is a mapping and refuses, naming the key-path, on a scalar or null where the template holds a mapping and on a mapping where it holds a scalar; two acceptance criteria cover both directions.
- F2 (list elements): D1 carries a present list whole, fills nothing inside its elements and still refuses on a key missing inside one; a criterion covers it.
- F3 (Reconcile note): D1 keeps keys the template lacks, a criterion covers it, and the Notes bullet names both `Reconcile` behaviours a reuse must avoid.
- F4 (Goal vs. one-time spawn): the Goal now allows one `lyx ide spawn` of each pre-landing worktree, the Auto-mode assumption states that only spawn regenerates `tasks.json`, and the Open risk names the prime whose #340 workaround D10 removes.
- F5 (liveness test): D2 defines alive as the `--if-absent` set, computed in the shared engine with no backend code, and a platform-independent fixture criterion covers the skip decision.
- F6 (tracked `tasks.json`): D4 runs `PathTracked` before any write and writes neither file nor exclude line when tracked; the criterion includes an absent `settings.json`.
- F7 (board exemption): D6 binds warp worktrees, the prime included, names `boardengine`'s `_board` writer as the one weft-side writer, and says it extends the junction bullet; Scope Out matches.
- F8 (D8 criterion): the acceptance criteria now carry the reconcile-wording check and the kept absent-file hint.

I spot-checked the code claims these fixes rest on.
`configengine.load` refuses on `MissingKeys` and keeps the `lyx config reconcile` hint on an absent file under `Load`.
`MissingKeys` treats a whitespace-only file as an empty document, so the "empty present file loads as the template" criterion is consistent with D1's ancestor rule.
`ideengine.SpawnDriven` holds the `PathTracked` + `ExcludeAnchoredDir` + Warn sequence D4 and D5 reuse.
`vscode.WriteConfig`'s interactive chain is the only `gitignore.Ensure` caller.
`aliveIDSet` is the present-and-not-dead set `classifyIfAbsent` decides against.

Attack-surface check: `--unless-name` (D2) can only suppress an add, and D2 states what counts as live and that it never removes, resizes or refocuses.
The D1 downgrade from refusal to fill is bounded to template key-paths with template values under mapping ancestors.
D4's overwrite is bounded to the untracked anchor `tasks.json`.
D5 and D6 remove a write rather than a guard.
Nothing in the record is something the writer's stencil told it not to gather, and no content needs relocation: the D2 mechanism narrative carries the stacking-not-window-size constraint the plan writer needs.
