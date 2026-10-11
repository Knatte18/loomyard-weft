---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4 (Bound paragraph)
    summary: D4's bound is false for per-worktree modules, whose local layer may set reed.shell, reed.tmux, reed.tmux_conf, shuttle.claude, shuttle.run_dir and the shuttle.claude_deny_* guards, untracked and invisible at landing
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (seed-only sentence), D1, D5, Acceptance criteria (reconcile row)
    summary: D2 has a seed-only module's upper layers pruned, which deletes the operator's own models aliases and burler lenses or fans that the template lacks, contradicting SeedOnly's never-rewrite contract
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6, R2
    summary: D6 does not state that the one-time strip turns a deliberate worktree value equal to the seeded default into an inherited one, which a later default change then silently alters
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D9 (removal sentence and Bound)
    summary: D9 does not name what its removal can discard, a hand-written local override placed in the pair while Seed-Child was Stuck, nor that this resume is the only window
---

### [BLOCKING] D4's bound does not hold for per-worktree modules

**Location:** `_lyx/discussion/decision-record.md` D4, the `Bound:` paragraph and the sentence "A per-worktree module's local layer may set any key."

**Issue:** D4's bound reads: "a local override can never change a key that runs a shell command (`darn.verify`, `landing.publish_verify`) or a hub-level fact shared across pairs".
This premise is false, because D4 lets the local layer of every per-worktree module set any key, and several of those keys run an executable, switch off a guard or reach across pairs:

- `reed.shell` is the program launched in each pane, and `reedengine` runs it directly (`exec.Command(e.cfg.Shell, ...)` in `internal/reedengine/proctree_windows.go`, and in the Selvage split).
- `reed.tmux` is the tmux executable.
- `reed.tmux_conf` is a tmux config file sourced into the hub server "on the boot that starts that server" (`internal/reedengine/template_posix.yaml`, `lifecycle.go`).
  The tmux server is shared by the hub, so this is a hub-level fact shared across pairs, and a sourced tmux config can run `run-shell`.
- `shuttle.claude` is the agent executable (`claudeBinary` in `internal/shuttleengine/claudeengine/command.go`).
- `shuttle.claude_deny_python`, `claude_deny_agent_tool` and `claude_deny_ask_user_question` are the standing PreToolUse guards (`standingDenies` in `claudeengine/settings.go`), so a local override can switch them off for every agent spawned in the pair.
- `shuttle.run_dir` "must stay worktree-LOCAL", because a root shared across worktrees makes the orphan sweep delete other worktrees' run directories (template comment).

Today these keys live in the tracked worktree file, so a change shows in the pair's diff, and `fabricengine.ReadConfigChanges` flags it at landing.
The local layer is untracked: nothing at landing shows that it exists, and no settings deny protects `.lyx/config/`.
Agents in the pair can write it (R3 admits this), so the second half of the bound, "everything else a pair can override comes from its board entry, whose history shows who set it", is also false.
D4 narrows the operator's ruling for darn and landing on exactly this shell-command ground (R1) but leaves the same exposure open in reed and shuttle, so the record is inconsistent on its own security premise.
The rubric's attack-surface question requires a true, stated bound for the new routing edge, and D4 has none for per-worktree modules.

**Fix:** Decide the per-worktree side explicitly.
One route: give per-worktree modules a `LocalKeys`-style denylist or allowlist too, which excludes at least the executable-path keys, the `claude_deny_*` guards, `tmux_conf` and `run_dir`.
The other route: keep "any key", but rewrite the bound so it names these keys, says that an untracked override of them is invisible at landing, and states what does bound it.
Either way, D5's refusal, D8's validation, the acceptance criteria and R3 follow the choice.
Correct the board-history sentence so it says board history covers only what Seed-Child writes, not later edits.

### [BLOCKING] Seed-only modules: pruning upper layers deletes operator-owned keys

**Location:** `_lyx/discussion/decision-record.md` D2 ("A seed-only module follows the same split: ... its upper layers are only pruned"), D1 (key-by-key merge), D5 (local layer refuses keys the template lacks), and the acceptance criterion "Reconcile adds missing keys only to the base layer, prunes retired keys and runs `Migrate` in every layer".

**Issue:** `configreg.Module.SeedOnly` marks a module whose key set is "open-ended and owned by the operator (e.g. models.yaml aliases, burler.yaml lenses/fans)".
`configsync` never rewrites a present seed-only file: it "neither adds nor prunes keys" (`internal/configsync/configsync.go`, the `m.SeedOnly` branch).
Neither `models` nor `burler` declares `OpenMaps`, so "retired" for them means any key the template lacks.
Under D2, reconcile prunes the upper layers of a seed-only module, so a pair's worktree `models.yaml` holding a custom alias, or a `burler.yaml` holding a custom lens or fan, loses it on the next reconcile.
D5 then refuses that same alias or lens in a local file, and D8 refuses it in `run_settings`, because its key path is absent from the template and sits under no open map.
D1's key-by-key merge also means an upper layer can never drop a lens from a fan or an alias inherited from the repo default.
The result is that the design deletes operator content that today's code protects, and the record never weighs this.
The acceptance criterion pins the pruning for every layer, with no seed-only exception.

**Fix:** State the seed-only rule in D2.
For example, seed-only upper layers are never pruned or migrated, and their top-level keys merge whole per entry, or the module's open-ended keys are treated as open maps across layers.
Align D5's unknown-key refusal and D8's key-path validation with that rule for seed-only modules, and add a seed-only exception or row to the reconcile acceptance criterion.

### [MEDIUM] D6 does not state what the one-time strip silently changes

**Location:** `_lyx/discussion/decision-record.md` D6 and R2.

**Issue:** The strip removes "every key whose value equals what the layers below resolve to" at the time of the walk.
A worktree whose value was a deliberate pin that happened to equal the prime's value, from which the repo default is seeded, loses the pin.
From then on, a later `--layer default` change alters that worktree's effective value with no change in its own files.
D6's rationale only addresses later walks ("never stripping a marked worktree keeps a later change to the default from erasing a deliberate override"), and the first strip has exactly that effect, which the Decision leaves unstated.
R2 names only the landing warning.
On the marker, the case of a pair forked mid-walk holds: a pair forked before its parent's strip commit lacks the marker and is stripped by the next walk, and a pair forked after it inherits marker and sparse files.
That part of the bound is sound.

**Fix:** State in D6 that the one-time strip turns every value equal to the seeded default into an inherited one, and that a later default change reaches it.
Name the bound, for example that the strip is reported per worktree and key in the reconcile output so the operator can restore a pin with `lyx config <module> --set`, and put that in an acceptance criterion.

### [LOW] D9's removal does not name what it can discard

**Location:** `_lyx/discussion/decision-record.md` D9, "It removes the local file of every registered module the run settings do not name, logging a Warn per removed file" and its `Bound:`.

**Issue:** Worktree-Create refuses a pre-existing branch (`internal/battenshed/create.go`), so a pair reaches Seed-Child fresh, except for a Seed-Child that went Stuck and is resumed with `lyx batten run <slug>`.
In that window, an operator who answered the D9 Stuck by writing the pair's override by hand with `lyx config --layer local` loses it on resume, with only a Warn in batten's log.
The Decision states that the result equals the run settings, but not this case or that the board entry is the only override that survives.

**Fix:** Add one sentence to D9 naming what the removal can discard: a hand-written local override placed in the pair while Seed-Child was Stuck.
It should also say that the D9 way forward, correcting `run_settings` on the board, is the only override that survives the resume.

## Other focus questions settled with no finding

- D8 and D9 premises hold.
  `configreg` imports `boardengine`, so `boardengine` cannot import `configreg`.
  Seed-Child reads the task's `recipe` fresh on every Call (`SeedChildDeps.ReadBoardType`, wired in `internal/battencli/wire.go`), and it writes the seed the child driver launches from, so no child driver exists while it runs.
- D1, D2, D5 and D7 read the same base layer, worktree file and sparse-layer rules for per-worktree and hub-wide modules, apart from the seed-only gap in F2.
- The lens-review dispositions in the support log that carry a requirement each appear in a Decision, Constraint, risk, acceptance criterion or Note.
  No relocation finding is warranted.
