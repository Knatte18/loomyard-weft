## Interview

Turn 1 (auto): read the board entry and the other board entries.
Sibling `twb-*` entries are identical bench copies of this task and are ignored; `fork-wait-noise` and `audit-reader-globs` touch neighbouring Webster areas and are kept out of scope.

Turn 2 (auto): explored the code at base `800a0e892` at module-boundary level.
Findings that shaped the record: `AwaitedShellPrefixes` already exists for `lyx webster recover-batch`; `Result.EndedShells` and the friction note already record a shell outstanding at run end; quarry v0.2.1 `statusForMatches` has no exported-only filter and is build-constraint-blind by design; `recover-batch`'s failed-digest path returns `output.Ok`; `loadRunWrites` audits Master sessions and each batch's current session id.

Turn 3 (auto): settled the decisions in the record.

Lens review (six forks, all returned):
- premises: gate re-prompt is shuttle's generic `gateRepromptText` that says fix the findings — fixed (decision 2 names it and supplies Webster text).
- premises, scenarios, goal-scope: no recovery session id is recorded at base, restart case — fixed (decision 3 adds the list, capture at spawn, old state loads empty, restart scenario).
- premises, goal-scope, scenarios: prefix match on shell label, and the gated fresh-arrival mechanism — fixed (decision 1 names both, label-shape scenario).
- goal-scope: unbounded hold below RunTimeoutMin — rejected: stated as a deliberate bound in decision 1 and Open risks.
- goal-scope: Decision 2 repeats a rule the stencil has — fixed by the root-cause finding above; the contradicting generic text is the change.
- goal-scope: #511 may end as "not fixed" — answered in decision 7: a quarry limit stays a labelled refusal, the Out list bars changing quarry.
- goal-scope: plan-time downgrade goes beyond DoneChecks — fixed (decision 8 states why).
- goal-scope, scenarios: `unix` tag, three candidates — fixed (decision 8, acceptance).
- scenarios: single candidate, collision, different units, later-card variant, malformed constraint — fixed (decision 8 bounds, acceptance).
- scenarios: failed-digest state save and sync, no respawn — fixed (decision 4, acceptance).
- goal-scope: Master ladder consistency for the non-zero exit — fixed (decision 4 bound).
- pattern-fit: completion-signal scan, leaf-packages, told-geometry, stencil-ownership, settings check for the named commands, old-state load — fixed (Constraints, acceptance).
- pattern-fit: refusal names a way forward — fixed (decision 4).
- pattern-fit: comment-line-breaks — fixed (Constraints).
- board-fit: `fork-wait-noise` and `proc-child-lifetime` dependencies, `parent-review-hardening` conflict risk — first two fixed in Constraints; `parent-review-hardening` rejected: a merge-text risk only, no design consequence.
- board-fit: `kick-start-pack`, `enclose-kickstart` reuse of the helper — rejected: unplanned reuse is not a requirement.
- leanness: Decision 9 and restating constraint bullets, assumption repeats, reproduction tree detail, tally in this log — fixed (decision 9 folded into Constraints, bullets trimmed, count dropped); the reproduction layout stays in decision 7 as its method.
- scenarios: ledger lists #511 cause open — kept: open until the Plan reproduces it.
- goal-scope: `docs/overview.md` — rejected: no module table or execution stack changes.

## Rejected alternatives

- Stopping the gate shell from lyx and recording its result discarded: shuttle has no pid for a provider shell and `lyx kills no shell` is documented behaviour; holding uses the existing prefix seam.
- A new `lyx webster` verb that spawns the verify fixer: a new verb for a guidance failure, with its own audit and spec surface; the message Master reads is the cheaper fix.
- A second recovery that starts from a clean tree with the first diff as reference: discards partial work the prompt already says to keep, and needs a new reset and diff-offer path.
- Having quarry honour build constraints: the resolver reports candidates by design so a caller never gets a silent pick, and the module is pinned outside this repo.
- Accepting any ambiguous answer in DoneChecks: would pass real duplicate declarations.
- Treating the exported-only hypothesis as fact for #511: unverified in quarry, so the record requires a reproduction.

## Review rounds

_No rounds yet._

## Question ledger

- Hold the turn end or stop the shell? Auto-pick: hold, via `AwaitedShellPrefixes`. Resolved.
- Where does the self-fix rule go? Auto-pick: into the gate re-prompt text Master reads. Resolved.
- Recovery attribution by batch or clean tree? Auto-pick: by batch. Resolved.
- Is the #511 cause unexported or test-file identifiers, or build-tagged pairs? Open; decision 7 reproduces first. Auto-pick: record both paths.
- Plan-time flag for variant members: warning or silent? Auto-pick: informational finding. Resolved.
- Which exit path of `recover-batch` returned 0? Auto-pick: digest-failed path; Plan confirms. Open.
