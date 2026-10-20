---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D4 (Decision and Bound)
    summary: D4 says a lenses.yaml started from the prime's copy takes burler's mappings, but under the no-overwrite bound a prime copy holding lenses or fans keeps them and burler's tuned values are dropped, a window D4's way forward does not name
    fix: In decision-record.md D4, change the Decision's parenthetical so only a target started from the template takes burler's mappings, and state that a target seeded from the prime's copy keeps whatever lenses or fans that copy holds, dropping burler's as kept-target. Extend D4's one-time-window sentence to name a prime-worktree-layer lenses.yaml (for example one written by lyx config lenses --set at the worktree layer) written before the first walk after deploy, with the same way forward. Add the prime-copy case to the kept-target acceptance criterion.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D2 (Bound, --set sentences)
    summary: D2 says --set knows any name under lenses and fans and that a name is one dot-free key, but never states the value shape --set takes for a new fan or where a path deeper than lenses.<name> or fans.<name> is refused, while today's open-map --set refuses a list value
    fix: In decision-record.md D2's Bound, add one sentence stating that under the per-name kind --set takes lenses.<name> as a string scalar and fans.<name> as a YAML flow list of lens names that replaces the fan whole, and that a key with a further dotted segment under either path is refused as unknown by Set's unknown-key check.
---

### [MEDIUM] D4's prime-copy start contradicts its own bound and opens an unnamed loss window

**Location:** `_lyx/discussion/decision-record.md` D4.

**Issue:** D4's Decision says that at the board's base layer an absent `lenses.yaml` "starts from the template (or the prime's copy under seeding)" and that "each burler key's mapping replaces the template default whole".
Its Bound then says "Only a target started from the template in the same pass takes burler's mappings".
The code agrees with the Bound, not the Decision: in `internal/configsync/movekeys.go`, `loadTarget` reads `primeConfig(seedDir, module)` for an absent target under seeding and leaves `fromTemplate` false, so `takeKey` treats any `lenses` or `fans` key in that copy as held.
Today such a held value equal to the template default is overwritten; D4 gives that overwrite up for these two moves, so burler's whole `lenses` or `fans` mapping is dropped as kept-target.
Ensure's walk and `lyx config reconcile` both seed (`SeedDefaults: true`), and the prime anchor is the prime worktree's anchor (`internal/configcli/hubwide.go`), so its copy is the prime's worktree-layer file.
D2 lets `lyx config lenses --set` write the worktree layer, so a prime sparse `lenses.yaml` holding one fan, written after deploy and before the first walk, makes the migration seed the board file from that sparse copy, drop the hub's tuned `burler.fans` whole, and fill the rest from the template.
The focus question was whether the migration keeps every lens and fan value: it does for a present board `lenses.yaml`, including one equal to the template (D4's bound holds there, and so does the Open risks claim that the old-binary alternation leaves `lenses.yaml` unchanged), but this prime-copy path loses burler's values in a window D4's stated way forward names only for a board file.

### [LOW] `--set` value shape and deep paths under the per-name kind are unstated

**Location:** `_lyx/discussion/decision-record.md` D2.

**Issue:** D2 says `lyx config lenses --set` knows any name under `lenses` and `fans`, and an acceptance criterion has it add a fan.
The only existing per-name `--set` route, the open map in `yamlengine.SetValues`, sets a scalar under the name and refuses a list ("a list or other shape is an error, never converted"), so a new fan cannot be written by analogy with it.
D2 also says a name is one dot-free key, but not what `--set lenses.a.b=x` does: whether Set refuses it or writes a nested mapping that `CheckLoad`'s strict decode then rolls back.
Both outcomes are bounded by `CheckLoad`, so nothing escapes, but the plan writer must pick the value grammar for a new path kind the record introduces.

## Focus departures

None.
The four directives were each reached:
D4 and D5's bound holds for a present board `lenses.yaml` and the Open risks claim holds (F1 covers the one start path where it does not);
D2 and D3 state both the later-template-change consequence and the template-entry-deletion consequence with their ways forward;
D2 places the per-name kind on the `lenses` module's `configengine.Spec` and states that a hub-only lens or fan is never warned as unread, backed by an acceptance criterion;
D2's `--set` edge bounds the layers (default and worktree, the local layer refusing the guard keys) and refuses an empty, over-length or undefined-lens fan through `CheckLoad`'s rollback, with F2 covering the value grammar.
