---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Acceptance criteria, second bullet, third sub-bullet)
    summary: The criterion says the rendered FRESH-READ, locate step and pack section equal the 1e39afc10^ fragment bodies with banners stripped, but the locate body carries the specs_dir marker and the pack-section body the pack_cards marker, so no rendered prompt can equal those raw bodies.
    fix: In decision-record.md's Acceptance criteria, replace the sub-bullet "its FRESH-READ, locate step and `## Kick-start pack` section text equal the pack-on fragment bodies at `1e39afc10^`, each banner stripped;" with two sub-bullets - "each new inline fragment's body, banner stripped, equals the matching pack-on fragment body at `1e39afc10^`, banner stripped;" and "the prompt's FRESH-READ, locate step and `## Kick-start pack` section equal those bodies with their markers filled (`specs_dir` with the deployed specs directory, `pack_cards` with the per-card subsections);".
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D4, and Scope Out's last bullet)
    summary: D4 makes begin-batch skip the pack-file write for the inline layout but does not state in the Decision what that lets through - a `<batch>-pack.md` from an earlier file-layout begin-batch of the same batch stays on disk - nor its bound; the bound sits only in Scope Out.
    fix: Append to D4's decision text, before its Rationale - "A `<batch>-pack.md` an earlier file-layout begin-batch of the same batch wrote stays on disk; the inline prompt never names it, as the pack-off prompt never does today, so the fork has no route to it." - and delete Scope Out's last bullet ("Cleaning up a `<batch>-pack.md` left by an earlier begin-batch …"), which that sentence then states once.
---

### [LOW] Text-equality criterion compares rendered text with raw marker-bearing bodies

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria, the `kickstart_pack_file: false` bullet.

**Issue:** `git show 1e39afc10^:contracts/stencils/webster/webster-fragment-locate-pack.md` ends with ``Card grammar is in `{{.specs_dir}}/loom/loom-plan-spec.md`.``, and the pack-section fragment at that commit ends with `{{.pack_cards}}`.
A rendered prompt holds the filled values, so read literally the criterion cannot pass, and a plan writer has to guess whether it means the fragment files or the rendered prompt.
The criterion is the layout's contract (D3 calls the fragments the measurement baseline), so it should say which comparison it pins.

### [LOW] D4's skipped pack-file write leaves its let-through outside the Decision

**Location:** `_lyx/discussion/decision-record.md`, D4 and Scope Out.

**Issue:** The discussion spec requires a Decision that introduces a switch or routing edge to state, in that Decision, what it can skip or let through and what bounds it.
D1 bounds the switch to forks with a built pack, and D2/D4 state what the inline layout skips (no pack file write, no `## Start here`).
What the skipped write lets through on a re-run, a stale `<batch>-pack.md` beside an inline prompt, and why that is harmless (no prompt names it) appear only in Scope Out and in the support log's rejected alternative.
The plan writer does read Scope, so this is a placement gap, not a lost requirement; moving it into D4 also keeps it stated once.
The flip criterion ("file to inline leaves a prompt that names no pack file") already matches this bound and needs no change.

## Checked and found sound

- D2/D3 placement: at `1e39afc10^` the body's `{{.kickstart_pack}}` marker ended the line "… do not continue to a later card on top of a broken one." directly before the blank line and `## The sandbox rule`, and `RenderForkPrompt` filled it with `"\n\n" + section`.
  D3's marker position and "the leading blank-line separation the existing fill uses" reproduce that, and an empty fill of either marker leaves no byte in both layouts, since each marker ends a line and brings its own separation.
- D3/D4 render paths: only `composeForkTemplate` and `composeRecoveryTemplate` read `webster-body-implementer` (`internal/websterengine/render.go`); D3 names both, and `ImplementerBodyTemplate`.
  `RenderRecoveryPrompt` today lists `kickstart_pack` as optional without a value, and D3 adds the new marker to that list.
- D2 premises: the webster stencil diff between `793c6a8ee` and `1e39afc10^` touches only the body and the master template; `1e39afc10` does not touch `webster-fragment-card-read-pack`, whose "which your first turn read" fits both layouts; the Edit-only rule is context in `1e39afc10`'s FRESH-READ diff, so it stood at `1e39afc10^`.
- D1: `configengine` fills template keys the merged stack lacks (`internal/configengine/config.go`, `yamlengine.FillMissing`), and `kickstart_pack` is `CategoryNone` in `ConfigSpec`.
- D5: every `logPack` call (`off`, `language-none`, `empty`, `on`) maps to one `pack_layout` value; the degraded-pack path (index error with a non-empty pack) logs `on` and so takes `file` or `inline`, and an index error with a nil or all-empty pack logs `empty` and takes `none`.
  The existing pins in `internal/websterengine/beginbatch_test.go` match `batch=… kickstart_pack=… resolved=… unresolved=…` as a substring, so a key appended after `unresolved` keeps them.
- Acceptance criteria: each Decision has a criterion that catches its failure, apart from F1's wording.
