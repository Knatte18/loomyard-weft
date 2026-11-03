---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md (D6 Bound and Prepare acceptance bullet)
    summary: Prepare warns on the worktree's .codex/config.toml, but in a linked worktree Codex takes the [hooks] table from the main checkout's .codex/config.toml, so the stated bound on committed hooks misses the file that runs
    fix: >-
      In D5 and D6, make Prepare's Warn name the `.codex/config.toml` of the main checkout root as well as the worktree's, for a linked worktree;
      D5 already reads that root through `internal/dotgit`.
      In D6's Bound, state the Codex behaviour with its source:
      in a linked worktree Codex removes the worktree layer's `hooks` table and inserts the main checkout's
      (`merge_root_checkout_project_hooks`, `codex-rs/config/src/loader/mod.rs`, tag `rust-v0.164.0`),
      while rules load only from the worktree's own `.codex/rules` (`ConfigLayerEntry::config_folder` in `codex-rs/config/src/state.rs` and `load_exec_policy` in `codex-rs/core/src/exec_policy.rs`, same tag), so the `.codex/rules/` refusal keeps checking the worktree alone.
      Widen the Prepare acceptance bullet so a main checkout holding a `.codex/config.toml` gets the Warn naming it, whether or not the worktree holds one.
  - id: F2
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md (Open risks first bullet, D6 launch-line paragraph)
    summary: The writable_roots setting is left untested although the Codex 0.164.0 source answers whether the config key is honoured, and a halt rule is handed to the build
    fix: >-
      Keep the parent's 2026-11-03 marking that the sandbox behaviour is proven in the build, and add to Constraints the part a desk study settles:
      `codex-rs/core/src/config/mod.rs` at tag `rust-v0.164.0` seeds the workspace roots from `sandbox_workspace_write.writable_roots` when no permission profile is selected, which is the `-s workspace-write` path the launch line takes.
      Check in the same source that a `-c` key reaches that config, and record the answer.
      Then narrow the Open risks line and D6's stop-and-return sentence to the one thing a source read cannot show, that the sandbox enforces the root on the linked `.lym` and `_lym` path.
  - id: F3
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md (D2 probe sentences, assist-record acceptance bullet)
    summary: A probe or resume of a round whose record predates the assistant fields has no stated reading, so it may fall back to the current config and reshape a live chair's table
    fix: >-
      Add to D2, after "A probe or resume of the round builds its table from the record, never from config edited since", that a record without the assistant fields reads as a round with no assistant, whatever the config now says.
      Add that case to the acceptance bullet on probe and resume from the recorded choice.
      The existing fan record already shows the fallback hazard: `readRecordedFans` in `internal/shedadapters/roundusage.go` errors on a record without its flag, and `Call` then falls back to the config's fan.
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md (D1 marker paragraph, D2 report paragraph, D6 Bound)
    summary: Where the marker lives and whether it is tracked is unstated, and D6 calls markers tracked content while the report sits under the never-tracked .lym
    fix: >-
      In D1 or D2, state that the marker and outputs sit under the anchor's `.lym` at the review path's subpath, as `PATTERN-durable-vs-ephemeral-state` requires for a never-tracked file, and that only the Go copy of a done report sits beside the review under `_lym`.
      In D6's Bound, drop "and markers" from the tracked content the writable weft reaches, or say which markers are tracked.
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/discussion/support-log.md (Interview, Turn 3)
    summary: Turn 3 says the Go copy beside the review answers the sandbox fact, which the parent's weft writable root now answers
    fix: >-
      Reword Turn 3's report clause so the weft checkout as a writable root (D6) answers the sandbox fact and the Go copy beside the review is named as the record keeper (D2).
---

Verdicts on the five focus items:

- Report path: D2, D6's launch line, Constraints, Acceptance criteria and Q5 all follow the parent's weft entry; no section names a `/tmp` directory, a kept copy or a macOS risk, and the remaining `/tmp` mentions are a Constraints fact and rejected alternatives. Turn 3 of the support log still carries the old reasoning (F5).
- Weft bound: D6's Bound states the edge and its limit, as the rubric's attack-surface question asks; it is stated by the Decision that adds the edge. The tracked-versus-untracked wording needs F4.
- Project rule files: D5 refuses a launch under `.codex/rules/` and an acceptance bullet covers it. The Codex source confirms the refusal checks the right directory, since rules load from each project layer's own `.codex` folder, never the main checkout's (F1 records the source). The hook half of that bound is incomplete (F1).
- Interrupted retry: D2 records the token before attempt 1 and rewrites `<round>b` before attempt 2, and builds the probe's whole table from the record. This settles the case the code shows, where `probeLiveReview` builds options from the bare round. One unstated case remains (F3).
- Instruction parity: D12 decides it, the Open risks hold no line a parent entry settles, and `lym heddle list` matches the Constraints bullet.

### [MEDIUM] Prepare's Warn checks the wrong .codex/config.toml for a linked worktree

**Location:** `_lym/discussion/decision-record.md`, D5 and D6 Bound, Prepare acceptance bullet
**Issue:** D6 accepts that automatic trust plus `--dangerously-bypass-hook-trust` runs any hook committed under the repository's `.codex/`, and names the Warn on a committed `.codex/config.toml` as part of what bounds it. The Warn is defined on the worktree's file only.
In Codex 0.164.0, a linked worktree's project layer has its `hooks` table removed and replaced by the main checkout's (`merge_root_checkout_project_hooks`, `codex-rs/config/src/loader/mod.rs`, tag `rust-v0.164.0`; `root_checkout_hooks_folder_for_dir` maps the directory to the main checkout's `.codex`).
A main checkout whose `.codex/config.toml` declares hooks, beside a worktree branch that holds no such file, runs those hooks outside the sandbox with no Warn.
Rules do not behave this way: `config_folder()` returns the layer's own `.codex` folder, so the refusal in D5 checks the right directory.

### [LOW] writable_roots left to the build although the source answers the config part

**Location:** `_lym/discussion/decision-record.md`, Open risks first bullet and D6 launch-line paragraph
**Issue:** The parent's entry of 2026-11-03 has the record mark the setting as to be proven in the build, so the marking stays.
The record also hands the build a stop-and-return rule on discovering that Codex ignores the key, and the rubric treats that as a deferred research question when a desk study can answer it.
`codex-rs/core/src/config/mod.rs` at tag `rust-v0.164.0` builds the workspace roots from `cfg.sandbox_workspace_write.writable_roots` when no permission profile is selected and the syntax is the legacy one, which is the `-s workspace-write` path.
The support log's Turn 2 shows the writer saw `writable_roots` in the binary and did not read the source.
What a source read cannot show is whether the sandbox enforces the root on the linked `.lym` and `_lym` path, so the build keeps that single check.

### [LOW] A record from before the assistant fields has no stated reading

**Location:** `_lym/discussion/decision-record.md`, D2 and the probe acceptance bullet
**Issue:** D2 says a probe or resume builds its table from the record, never from edited config. A round started by a binary that predates the assistant fields has a record without them.
The fan record's reader errors on a missing flag and `Call` then takes the current config's fan (`internal/shedadapters/burlerreview.go` around the probe fan selection), so the same fallback for the assistant would add an assistant seat to a chair spawned without one.
D2 does not say which way a missing field reads.

### [LOW] Marker location and tracked status unstated

**Location:** `_lym/discussion/decision-record.md`, D1, D2 and D6 Bound
**Issue:** D1 derives the marker and outputs "from the review path", which sits under `_lym`, while D2 puts the report under `.lym`. D6's Bound lists "other rounds' review files and markers" as tracked content the run's commits carry.
`PATTERN-durable-vs-ephemeral-state` puts a never-tracked file under `.lym` at the same subpath, so a marker belongs there, and then the Bound's "tracked" claim is wrong for markers.
The record does not say which side the marker takes.

### [LOW] Stale reasoning in Turn 3 of the support log

**Location:** `_lym/discussion/support-log.md`, Interview, Turn 3
**Issue:** "the report under `.lym` with a Go copy beside the review (D2) answers the sandbox fact" predates the parent's weft entry. The writable root answers the sandbox fact, and the copy serves the round's records, as D2 now says.
