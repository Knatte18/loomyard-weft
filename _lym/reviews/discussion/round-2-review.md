---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D2 (report under /tmp, kept copy under .lym), D6 launch line, Constraints (.lym and _lym bullet), Open risks third bullet, Acceptance criteria D2 bullets
    summary: D2 still writes the assistant's report in a /tmp directory and copies it into .lym, against the parent's Added-after entry making the weft checkout a Codex writable root so the report goes under the anchor's .lym as a Claude seat's does
    fix: >-
      Rewrite D2 to follow the Added-after entry "Weft checkout as a Codex writable root" of 2026-11-03.
      The assistant writes its report directly under the anchor's .lym, at a path derived from the review path and the attempt token (D1).
      Drop the /tmp directory, its 0700 mode and hash name, and the kept copy, which no longer has a reason to exist.
      The done marker is written once the seat gate has passed.
      In D6, add the weft checkout to the launch line as a writable root, the setting named and marked as proven in the build as the entry says.
      Change the Constraints bullet on .lym and _lym so that it says they resolve into the weft checkout, which D6 adds as a writable root,
      and no longer says Codex cannot write there.
      Delete the third Open risks bullet (macOS /tmp).
      Update the D2 acceptance criteria, which still name a report path under /tmp and a kept copy, and update Q5 in the support log's question ledger.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D6 Bound (weft checkout as a writable root)
    summary: Making the whole weft checkout writable is a new edge, and no Decision states what the session can then write there or what bounds it
    fix: >-
      In D6's Bound, state what the weft writable root lets through.
      The session can write anywhere in the weft checkout, not only its report.
      That includes the run's _lym config, the shed status files, the discussion and plan records, and other rounds' review files and markers, tracked content the run's commits carry.
      State what bounds it: only the assistant's prompt, as for a Claude seat, plus the seat gate on its own report.
      If a narrower root is chosen instead, such as the assistant's per-round directory under .lym, state that root and drop the rest.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1 and D2 (recorded attempt token, probe and resume), Acceptance criteria bullet on probe of a retried round
    summary: The attempt token is recorded with the fans, which are written once before the attempt loop, and the chair's probe always uses the bare round, so an interrupted retry's chair and assistant are never found
    fix: >-
      In D2, state that the review row rewrites the round's record with the attempt token (<round>b) before attempt 2 spawns.
      State that probeLiveReview builds the whole table, chair included, from the recorded token rather than from the bare round number.
      A probe after lym dies mid-retry then finds the retry's chair and its Codex assistant, and stops or resumes them.
      Keep the acceptance criterion and extend it to the chair.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (automatic trust), D6 Bound, Constraints bullet on -a never -s workspace-write
    summary: Automatic trust also loads a trusted project's committed .codex/rules allow rules, which run commands outside workspace-write, and neither D6's bound, the Warn nor the Constraint says so
    fix: >-
      Correct the Constraint: Codex 0.164.0 loads rules/*.rules from the home and from each trusted project layer's .codex/ folder
      (codex-rs/core/src/exec_policy.rs, load_exec_policy, tag rust-v0.164.0), and the interactive TUI cannot turn this off (spike, Approvals and sandbox).
      In D6's Bound, state that an allow rule committed under .codex/rules/ in a trusted root lifts the sandbox for its command prefix, so the session can then write anywhere and reach the network.
      Pick the bound and state it.
      Either Prepare refuses to launch in a worktree or main checkout holding .codex/rules/, with a way forward naming the directory,
      or Prepare's Warn names .codex/rules/ beside .codex/config.toml and D6 accepts the exposure explicitly.
      Add the chosen behavior to the D5/D6 acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D12, D2 (context the chair gets), Open risks second bullet
    summary: By default the Codex seat gets none of the heddle text the Claude chair loads from CLAUDE.local.md, against the operator rule that a Codex reviewer always gets the chair's context including the repository's agent instructions
    fix: >-
      Decide in D12 how the assistant receives, by default and with no operator step, the repository instructions the chair receives.
      For example, the codex-file render could also carry repository-layer entries that target claude-file, or the assistant's stencil could point it at the anchor's CLAUDE.local.md.
      State whether the shipped entries are included and why, citing the board entry's operator rule of 2026-10-27.
      Then drop the Open risks bullet on `lym heddle set nicknames`, and update the D12 acceptance criterion.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md Open risks first bullet; _lym/discussion/support-log.md Question ledger Q4
    summary: The parent's Added-after entry on the opt-in Codex subagent fan settles the first Open risk and Q4, yet both still stand as open
    fix: >-
      Delete the first Open risks bullet (fan before Step 4).
      Mark Q4 in the support log's question ledger Resolved, citing the parent's Added-after entry of 2026-11-03.
  - id: F7
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (creates any missing link), D13 (logout removes the link)
    summary: After a logout, a login in the lym home leaves a regular auth.json that Prepare neither replaces nor reports, so the lym home's token silently diverges from the operator's
    fix: >-
      In D5 or D13, state that Prepare checks that the home's auth.json and sessions are links to the operator home's entries.
      On a regular file or a link elsewhere, Prepare either logs a Warn naming the path and the way forward (remove it so Prepare relinks) or refuses the launch.
      Add that case to the D5 acceptance criterion.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 Bound, D3 (assist_reviewer_timeout_min)
    summary: Nothing keeps the assistant's start, timeout and one await cap below the chair's review_timeout_min, so a large assist timeout lets the chair time out inside its wait and the assistant gates the round
    fix: >-
      In D3, have config validation refuse an assist_reviewer_timeout_min not below review_timeout_min, naming both keys,
      or have the table builder cap the assistant's timeout below the chair's.
      State the rule in D1's Bound and add it to the D3 acceptance criterion.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D1 (marker texts), D2 (seat gate)
    summary: The marker names no text for an assistant that ends done with its review-parse gate failed, and let the run through is undefined
    fix: >-
      In D1, add the marker text for a done run whose seat gate failed (for example gate failed), and state that such a report is not copied beside the review.
      Define let the run through as the gate's own pass-through case, or drop the phrase.
      Extend the D1 acceptance criterion with that ending.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D12 second paragraph
    summary: D12 claims the codex-file render carries the anchor's instruction files alone while no entry targets it, but fileText renders no file when no entry names the target
    fix: >-
      Replace the D12 sentence with what fileText in internal/heddleengine/render.go does.
      While no entry targets codex-file, no AGENTS.override.md is rendered,
      and Codex itself loads the anchor's AGENTS.md, or CLAUDE.md through project_doc_fallback_filenames.
  - id: F11
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/support-log.md Interview Turn 4
    summary: The support log says shuttle retries a first turn that never starts, contradicting D6
    fix: >-
      In Turn 4, change which shuttle retries to which shuttle does not retry, matching D6.
---

The parent recorded "Weft checkout as a Codex writable root" while this review was being written.
It replaces the entry that put the report in the warp tree or under TMPDIR, so this review judges D2 against the new entry.

The four focus questions, settled:

- **Report path writable (D2).** Under the new entry, the report goes under the anchor's `.lym`, which is writable once the weft checkout is a writable root.
  The parent marks that setting as proven in the build, so this review does not flag it as deferred research.
  D2 still follows the replaced design (F1), and the new edge carries no bound (F2).
- **Stale per-round files across attempts (D1, D2).** A fresh retry is safe: the `<round>b` token gives the retry its own marker, and stale files are removed before the chair starts.
  A probe or resume of an interrupted retry is not safe (F3).
- **Bounds on D5, D6 and D13.** The `auth.json` write path is researched and correct: Codex `rust-v0.164.0` `codex-rs/login/src/auth/storage.rs` saves with `truncate(true).write(true).create(true)` and renames nothing.
  The trust edge leaves out project rule files (F4), and nothing states what happens after a logout and then a login (F7).
- **Instruction parity (D6, D12).** Both shipped entries and the repository entry `nicknames` target only `claude-file` (`lym heddle list`), so the Codex seat gets none of them by default (F5).

### [BLOCKING] D2 contradicts the parent's weft-writable-root decision

**Location:** `_lym/discussion/decision-record.md` D2, D6, Constraints, Open risks, Acceptance criteria

**Issue:** The newest Added-after entry adds the weft checkout to the Codex sandbox's writable roots, so the assistant writes its report under the anchor's `.lym` as a Claude seat does.
That entry replaces the earlier one about a warp-tree or TMPDIR path.
D2 still writes the report in a `/tmp` directory lym creates "with mode 0700", then keeps a copy under `.lym`.
The Constraints bullet still says Codex "cannot write under the anchor's `.lym` or `_lym`", and an acceptance criterion still names "its report path under `/tmp`".
A plan writer reading the record gets two contradicting report locations.
The `/tmp` directory also breaks `PATTERN-lym-temp-root` ("never directly in the system temp directory"), with a predictable name and no ownership check or removal.
Following the parent's entry removes that too.

### [MEDIUM] Weft checkout as a writable root has no stated bound

**Location:** `_lym/discussion/decision-record.md` D6 Bound

**Issue:** A writable root at the weft checkout opens all of it to the session, not only the report path.
The weft holds the run's `_lym` content: config, shed status, discussion and plan records, and every round's review files.
Today, D6's Bound says `workspace-write` "lets the session write anywhere in the worktree", which no longer describes the reach.
Under the rubric's attack-surface rule, the Decision that adds the edge must state what it lets through and what bounds it.

### [MEDIUM] An interrupted retry's chair and assistant are never probed

**Location:** `_lym/discussion/decision-record.md` D1, D2, Acceptance criteria (probe of a retried round)

**Issue:** D2 records the attempt token "beside its recorded fans".
`recordRoundFans` runs once, before the attempt loop (`internal/shedadapters/burlerreview.go:229-233`).
The `b` token is minted inside the loop (`burlerreview.go:254-257`).
`probeLiveReview` sets `opts.Round = strconv.Itoa(round)` (`burlerreview.go:413-418`), and the Round names every run directory.
If lym dies during attempt `<round>b`, the probe looks for `<round>` and finds no live chair.
The `<round>b` Codex strand is then neither found nor stopped, and it spends capped Codex budget with no lym process enforcing its timeout.
The acceptance criterion that a probe of a retried round finds the retry's assistant cannot be met under D2 as written.

### [MEDIUM] Automatic trust loads committed .codex/rules allow rules that lift the sandbox

**Location:** `_lym/discussion/decision-record.md` D5, D6 Bound, Constraints (`-a never -s workspace-write`)

**Issue:** Codex `rust-v0.164.0` `load_exec_policy` (`codex-rs/core/src/exec_policy.rs`) collects `rules/*.rules` from every User and Project layer's config folder.
It skips them only under `ignore_user_and_project_exec_policy_rules`, which only `codex exec --ignore-rules` sets; the spike found the TUI rejects that flag.
The spike shows an allow rule runs its command outside `workspace-write`.
D5 trusts every launch's worktree and main checkout without a prompt.
So a committed `.codex/rules/` allow rule for `bash` lets the assistant write anywhere and reach the network.
The Constraint implies the home's rules are the only exemption source, and D6's Bound and Prepare's Warn name only `.codex/config.toml`.

### [MEDIUM] Codex seat misses the chair's instruction text by default

**Location:** `_lym/discussion/decision-record.md` D12, D2, Open risks (second bullet)

**Issue:** The board entry records "A Codex reviewer always gets the context a Claude reviewer of the same round gets, the PATTERN overview and the repository's agent instructions included (operator, 2026-10-27)", and D2 claims that parity.
`lym heddle list` shows `lym-batching`, `lym-plan-edits-before-gate` and `nicknames` all target `claude-file` only.
The repository holds no `CLAUDE.md` or `AGENTS.md`.
So the chair loads all three through `CLAUDE.local.md`, while the Codex seat gets none of them.
D12 leaves parity to an operator `lym heddle set`, recorded as an Open risk, so the operator rule is unmet until someone acts.
How the assistant gets these instructions is the writer's own choice and belongs in D12.

### [LOW] Settled fan question still open

**Location:** `_lym/discussion/decision-record.md` Open risks (first bullet); `_lym/discussion/support-log.md` Question ledger Q4

**Issue:** The parent's Added-after entry "Opt-in Codex subagent fan allowed before Step 4" says "Drop the Open risk and close Q4".
The record still carries the first Open risks bullet, and Q4 still reads Open.

### [LOW] Regular auth.json after a login in the lym home

**Location:** `_lym/discussion/decision-record.md` D5, D13

**Issue:** D13 relies on Prepare relinking `auth.json` after a logout.
A login in the lym home before the next Prepare creates a regular file, and `internal/fslink`'s `prepareLink` refuses to clobber an existing path.
D5 creates only missing links, so the lym home keeps a token apart from the operator's, and nothing reports it.

### [LOW] Assistant timeout not bounded below the chair's

**Location:** `_lym/discussion/decision-record.md` D1 Bound, D3

**Issue:** D1's wait bound is the assistant's start, its timeout and one await cap.
`review_timeout_min` defaults to 240 and `assist_reviewer_timeout_min` to 60, but nothing relates the two keys.
An operator who sets the assist timeout at or above the chair's makes the chair time out in its wait, so the assistant decides the round, against the Goal.

### [LOW] No marker text for a failed seat gate

**Location:** `_lym/discussion/decision-record.md` D1, D2

**Issue:** D1 names the endings `done`, `died`, `timeout`, `stopped`, `not started` and the wait error.
An assistant whose review-parse gate fails ends done with its gate failed (`internal/seatengine/doc.go`), and no marker text covers that ending.
"Let the run through" is not defined either, so a malformed report the gate lets through might be marked `done`.

### [LOW] D12 misstates the codex-file render with no entry

**Location:** `_lym/discussion/decision-record.md` D12

**Issue:** `fileText` in `internal/heddleengine/render.go` returns false, and renders nothing, when no entry names the target.
D12 says the render "carries the anchor's instruction files alone until one does".

### [NIT] Support log contradicts D6 on the start retry

**Location:** `_lym/discussion/support-log.md` Interview, Turn 4

**Issue:** Turn 4 says a turn that never starts "fails the start, which shuttle retries".
D6 and `internal/shuttleengine/doc.go` say it is not retried.

## Rejected

- interactions F5 (macOS `/tmp` writability left as an Open risk): moot under the parent's weft-writable-root entry, which puts the report under `.lym`; F1 drops the bullet.
- interactions F3 (D2's `/tmp` against the parent's "under TMPDIR"): the parent replaced that entry; the conflict with the new entry is F1.
- bound-breaker 2 (predictable `/tmp` name, no ownership check, leak): moot once D2 follows the new entry; noted in F1.
- walkthrough F1: the same defect as existing-paths F1, kept as F3.
- walkthrough F2 and interactions F2: the same defect, kept as F5.
- interactions F6: the same defect as bound-breaker 1, kept as F4.
- interactions F4: the same defect as bound-breaker 3, kept as F8. The walkthrough found it held at the defaults, so it stays LOW.
- interactions F7: the same defect as bound-breaker 4, kept as F7.
- bound-breaker 5: kept as F11.
