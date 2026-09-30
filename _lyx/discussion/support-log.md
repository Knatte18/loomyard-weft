# Support log — batten-open-ide

## Interview

No operator was present; the session ran autonomously, so each turn below is a question put to the codebase and the pick made from it.

1. **Where is the strand-is-up point?**
   `battenshed.InnerRunDeps.Spawn` runs `lyx loom start --no-attach` in the task worktree and returns once the child's driver is up (its own doc comment says so).
   `Seed-Child` only writes, commits and pushes the seed.
   Pick: `Run-Shed`'s spawn.
2. **A new recipe row, or a closure inside `Run-Shed`?**
   The brief asks for "a batten producer after the child's driver strand is up".
   No row can sit after the spawn and before the watch without splitting `Run-Shed`.
   Pick: an injected closure called by `InnerRun` after a successful spawn.
3. **How often does it fire?**
   `Run-Shed` spawns again on an approved `awaiting` child and on a running child without a confirmed spawn.
   Pick: once per run, gated by a scratch marker written after the attempt.
4. **What does the driven `folderOpen` chain run?**
   `reed attach` self-boots a cold session, and `loom start` brings reed up before the driver spawns.
   Pick: one `lyx reed attach` task on `folderOpen`.
5. **Is an existing `tasks.json` rewritten?**
   The brief left this open.
   Pick: the driven path overwrites `tasks.json` and keeps `settings.json`; plain `ide spawn` keeps write-only-when-absent.
6. **Does `ide spawn` itself learn to detect a driven pair?**
   Detection would mean reading the pair's seed, and a manual `ide spawn` followed by `loom start` is today's intended interactive workflow, where the bare claude strand is the operator's own.
   Pick: no; batten asks for the driven chain explicitly.
7. **Does the IDE config write dirty the task worktree?**
   `vscode.WriteConfig` calls `gitignore.Ensure`, which appends a `# === lyx-managed ===` block to `.gitignore` unless one already carries the entry.
   This repo's `.gitignore` ignores `**/.vscode/` inside a mill-managed block, not a lyx-managed one, so `Ensure` would still edit it.
   Pick: the driven path edits no tracked file and uses `.git/info/exclude`.
8. **Does the chain depend on the child's driver (`go` vs `llm`)?**
   Pick: no; either driver's session already has its driving surface, and reading the driver would breach the Driver Choice Single-Site Invariant.
9. **Is an opt-out needed?**
   Pick: no; YAGNI, and the headless degrade covers CI.
10. **How are tests kept from opening windows?**
    Pick: stub the `InnerRunDeps` closure in `battenshed` tests and `ideengine.CodeLauncher` wherever the real closure is reached.

## Rejected alternatives

- **An `Open-IDE` row between `Seed-Child` and `Run-Shed`, relying on `reed attach` to boot the session.**
  The window would open before the driver strand exists, contradicting the brief, and the IDE's attach would race `loom start`'s own `reed up` for the session boot.
- **Splitting `Run-Shed` into `Start-Child` → `Open-IDE` → `Run-Shed`.**
  `Run-Shed` must keep its own re-spawn logic for resume and approval, so the spawn-confirmation marker and idempotence would live in two rows;
  it also changes the recipe's durable row set for no behavioural gain over a closure.
- **Opening on every spawn.**
  Re-opens a window the operator closed and repeats the headless warning on each approval resume.
- **Retrying a failed open on the next spawn.**
  Same repeated-warning cost on a headless machine, for a case (transient `code` failure) with a trivial manual remedy (`lyx ide spawn <slug>`).
- **Keeping write-only-when-absent for the driven `tasks.json`.**
  A leftover interactive chain would bring back the second bare claude strand.
- **Making plain `ide spawn` detect a driven pair from its seed.**
  Reads run state from an unrelated module and changes an operator-facing command's behaviour that the brief did not ask to change.
- **Choosing the chain by the child's driver.**
  A second reader of the recorded driver is barred by the Driver Choice Single-Site Invariant, and both drivers want the same chain.
- **An `open_ide` recipe config key or `--no-ide` flag.**
  No stated need; the warning-only failure covers headless use.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — where the strand-is-up point lies.** Resolved (auto-pick): `Run-Shed`'s spawn; `Seed-Child` starts nothing.
- **Q2 — row or closure.** Resolved (auto-pick): injected `InnerRunDeps` closure called by `InnerRun`; no recipe change.
- **Q3 — firing frequency.** Resolved (auto-pick): once per run, scratch marker written after the attempt regardless of outcome.
- **Q4 — driven `folderOpen` chain.** Resolved (auto-pick): `lyx reed attach` only.
- **Q5 — rewrite an existing `tasks.json`.** Resolved (auto-pick): driven path overwrites `tasks.json`, keeps `settings.json`; plain `ide spawn` unchanged.
- **Q6 — driven-pair detection in `ide spawn`.** Resolved (auto-pick): none; the caller chooses the chain.
- **Q7 — `.gitignore` dirtying the task worktree.** Resolved (auto-pick): driven path uses `.git/info/exclude`, never a tracked file.
- **Q8 — chain depends on driver.** Resolved (auto-pick): no.
- **Q9 — opt-out knob.** Resolved (auto-pick): none.
- **Q10 — test isolation from real VS Code.** Resolved (auto-pick): stub the closure and `ideengine.CodeLauncher`.
- **Environment note:** `lyx board get` printed a valid `{"ok":true,"task":{...}}` envelope but exited with status 2. The task was read correctly; not repaired, reported here for the operator.
