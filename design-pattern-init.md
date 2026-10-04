# PATTERN for loomyard, manifest/ retired, plus repo conventions

`pattern-init` · Tasks · feature · running · Webster-Burler

## Goal

loomyard gets a repo-root `PATTERN.md` so lyx agents receive the constraints directive, which no run has received so far; CONSTRAINTS.md moves there, as a thin overview of one-line `PATTERN-*` entries with background files for those that need them (format below).
`manifest/` is retired in the same pass, since both sweeps repoint the same docs, stencils and CLAUDE.md.
Runs alone, last: it repoints CONSTRAINTS.md in ~130 files.

## Also folded in

- `fabric-vocabulary-everywhere`, `model-spec-short-form`, `ide-hub-workspace-followups`, `agent-role-constants` and `manifest-retire` below.
- `manifest-retire`'s Someday strike list was decided by the operator on 2026-10-03 (below).
- `model-spec-short-form`'s model trial (`fable[high]` for discussion and review) moved to `convergence-plan`, where it is measured; this task does only the short form, the dead link and Master's explicit effort.

## From `pattern-init`: Initialize PATTERN for loomyard: CONSTRAINTS.md becomes _lyx/PATTERN.md

Operator 2026-10-01: CONSTRAINTS.md is millhouse's setup; loomyard's variant is PATTERN, not yet initialized.
Verified: no _lyx/PATTERN.md in loomyard-weft, and there is no init verb. internal/pattern injects a role directive (implementer, review-fix, orchestrator) pointing at _lyx/PATTERN.md into webster fork/Master, burler and loom plan prompts only when the file exists, so today's runs got no PATTERN directive (they only saw CONSTRAINTS.md through CLAUDE.md and stencil mentions).

### PATTERN format (operator, 2026-10-04)

- PATTERN lives in warp, not weft (operator, 2026-10-04): weft is the agents' scratchpad and every weft branch is deleted, while PATTERN is binding guidance for the code, changed in the same commit as the code it governs and landed by the ordinary merge.
- `PATTERN.md` at the repo root is a thin overview: one bullet per entry, grouped under a few topic headings.
  Each bullet is one line: the entry's name, then a one-liner that says both when the entry applies and what it requires, written so an agent can tell from the line alone whether the entry matters to its task.
- An entry that needs more than a line gets a background file, linked at the end of its bullet; an entry without a link is complete as written.
  An agent reads a background file only when it judges the entry relevant.
- Entry names are `PATTERN-<kebab-name>`, e.g. `PATTERN-paths-passed-in`: one word in a terminal and in `@` references, a valid filename everywhere, and recognisably not a board slug.
  The `<kebab-name>` part is chosen to read clearly, not copied from CONSTRAINTS.md's heading.
- A background file is `pattern/PATTERN-<kebab-name>.md` at the repo root (operator: `docs/` is supporting literature, not binding guidance), named exactly as its entry, so one grep finds the entry, the file and every citation.
- Every agent gets the whole overview inlined into its prompt, not a pointer: it is guaranteed seen, saves a read turn per agent (each Webster fork included), and stays cheap because it is thin and cache-stable; there is no per-task or per-card selection.
  Each directive stencil in `contracts/stencils/pattern/` keeps its instruction section (binding, wins over the rest of the prompt, violations are blocking) and gains a marker the overview's content fills, replacing its "Read _lyx/PATTERN.md in full" bullet; background files stay pointers.
  This lifts the stencils' marker-free rule and changes `internal/pattern`'s doc on why the pointer stays relative.
  Add the directive also to Discussion-Write and the Bouncer judge, which get none today.
- The format test also caps the overview's size, so content that outgrows a line moves to a background file.
- PATTERN's text files carry no tie to the tooling that develops the repo (operator, 2026-10-04): rules for the code only, never text addressed to agents or describing the dev workflow; that wording stays in the stencils and `internal/pattern`.
- An entry enforced by `go test` says so in its bullet, e.g. a trailing `(test)`.
- A format test pins: names unique and matching `PATTERN-[a-z0-9-]+`, each entry one line, every link resolving, and every file under `pattern/` having an entry.
- Citations (review findings, decision records, code comments) use the entry name.

### Work

Move CONSTRAINTS.md's invariants into that format; `internal/pattern` resolves the files from the warp worktree root, not `_lyx/`; repoint everything that names CONSTRAINTS.md: CLAUDE.md, stencils (loom discussion/plan/rework, webster master and prefix-recovery), the ly-drive skill, contracts/specs, burlercli/run.go, loomshed and treadleengine docs, and contracts/stencils/citation_enforcement_test.go; decide whether a `lyx pattern init` verb is wanted for other repos.

## From `fabric-vocabulary-everywhere`: Fabric vocabulary: warp and weft only inside fabric, also in docs, specs and tests

Operator 2026-09-30: warp and weft are fabric-internal words; nothing outside fabric should know them.
Seen on refusal-audit: contracts/specs/refusal-spec.md and several _test.go files outside fabric use them, and the scan let them through.
Want: extend the scan to contracts/, docs/, manifest/, plugins/ skills and test files outside the owner set (or an explicit allowlist for fabric docs), and sweep the existing hits.

## From `model-spec-short-form`: Model specs in the short form everywhere, and per-role model choice revisited

Seen 2026-10-01: long form in internal/loomengine/template.yaml (discussion, plan, review, friction), internal/websterengine/template.yaml (recovery), internal/landingshed/template.yaml (conflict, describe) and contracts/specs/llm-model-spec.md's examples; the spec only mentions in one sentence that a bare bracket token means effort. Template comments cite docs/reference/model-spec.md, which is missing; the spec lives at contracts/specs/llm-model-spec.md.
Webster master is plain `sonnet`, so its effort (medium) comes invisibly from models.yaml; write it explicitly.
Done on `main` 2026-10-04 (`08a365ff9`, `e13537cee`): the templates carry short-form specs with explicit effort.
Left: contracts/specs/llm-model-spec.md's examples, the dead docs/reference/model-spec.md link in template comments, and the long-form specs in internal/landingshed/config_test.go's malformed-input fixtures.
Model choice (operator, 2026-10-01): try `fable[high]` for discussion and review, where design judgement matters (refusal-audit's holes came from opus Discussion and opus reviews; a Fable review found the design problem), keep sonnet for implementation; change it only between runs and measure review rounds, PR rejects and interventions before and after (convergence-plan).

## From `ide-hub-workspace-followups`: ide hub workspace: review follow-ups

Follow-ups from the PR #301 review (`85902381e`), all non-blocking:
- `internal/fabricengine/hubworkspace.go` builds the prime path by hand (`filepath.Join(l.HubPath, primeName, l.AnchorRel)`) instead of `WorktreePath`.
- `internal/ideengine/spawn.go` now WARNs on every spawn whose prime resolution fails, task spawns included (silent before).
- Removing the last pair prunes `_portals/<anchor>`, so the hub workspace shows a missing folder until the next prime spawn.
- Relative-path settings spliced from the prime into the workspace file resolve against `_launchers/`, not the prime.

## From `agent-role-constants`: each module names the agents it spawns with its own constant

Operator 2026-10-02: role names stay owned by the module that spawns the agent; no central list, since a new module with its own agent must touch no shared file, and `agentname`'s role set stays open.
Seen: most spawn sites pass their role as an inline literal (`driver` in `loomcli/driverspec.go`, `discussion`/`plan`/`rework` in `loomengine`, `burler`, `friction`, `describe`, `conflict`, `judge`/`triage`/`targeting` in `treadleengine`).
Want: each spawn site takes its role from a named constant in its own module, and the Agent Name Invariant states that the spawning module owns its role.
Webster's own `master` → `webster` rename is in `webster-verify-gate`.


## From `manifest-retire`: retire manifest/ into bolt and package docs

Retire `manifest/`: first bring roadmap.md's surviving Someday items into bolt in the new tier format, then move the durable reference content into package docs, fix inbound links and delete the folder.
Millhouse has no ownership of `manifest/` (operator, 2026-10-02).

### Someday items

Not yet in bolt, to bring in as notes: `raddle`, `hardener`, `webster-parallel-execution`, `reed-multi-window`, `warp-visibility`, `baseline-drift-gate`, `config-overlay`, `fabric-reset-to-sha`, `finalize-discrepancy-doc`, `shedrecipe-capabilities`, `doctor`, `session-sync`.
The proposed strikes were decided by the operator on 2026-10-03 and are already on the board as notes: `fabric-windows` (Windows support stays, so CLAUDE.md's fslink rule stays), `semantic-index`, `issue-intake`, `ide-opt-in`, `plugin-packaging`, `burler-round-models` (was the per-round provider selector), `fabric-monorepo-verbs`, `reed-slack-relay`.
Struck: the shuttle tools-restriction (replaced by note `agent-permission-setup`) and bulk-mode clusters (replaced by note `codex-engine`).
Remove these items from `manifest/roadmap.md` with the rest; keep `manifest/designs/` docs only for designs not yet built, per CLAUDE.md.

### manifest/ retirement

The triage was done in the bolt review of 2026-10-01/02; what survives is already in bolt.
This task moves the durable reference content into package docs, fixes inbound links and deletes the folder (Millhouse has no ownership of `manifest/`).

- `manifest/HANDOFF.md`: the hub run loop goes to the `internal/orchcli` package doc; the rest is spent.
- `hardener.md`: the text-review vs behaviour-review distinction and the respawn rationale go to `internal/treadleengine`'s doc.
- `fabric-windows-verification.md`: the Linux-only caveat goes to the `internal/fabricengine` and `internal/fslink` docs.
- `webster-parallel-execution.md`: the 42-card case study and the index-race rationale go to `internal/websterengine`'s doc.
- `review-finding-classification.md`: the convergence case study goes to `internal/burlerengine`'s doc.
- `curation-triage.md`: one line in the boardengine doc ("anyone adds notes, the orchestrator curates"); the automation is struck.
- Inbound links: `internal/treadleengine/doc.go`, `internal/fabricengine/livestate_doc_test.go`, `contracts/specs/loom-plan-spec.md`, `docs/overview.md` (lifecycle text and design links), `crucible/README.md`, `crucible/review-prompt-template.md`, CLAUDE.md's Task completion rule, `.claude/agents/mill-done-gate-fixer.md`.


### From `manifest-to-bolt`: Move everything in manifest/ into bolt and retire the folder

Operator 2026-10-01: LYXHUB takes over all development of loomyard; everything in manifest/ goes into bolt.
The operator also says much of it is outdated: triage each roadmap Someday item with the operator (keep as tier 3 or tier 2, merge, or strike) and carry its designs/<name>.md over as the bolt entry's design doc.
Afterwards: delete manifest/, and update CLAUDE.md (Task completion section names manifest/roadmap.md and manifest/designs/), docs/overview.md's Documentation Lifecycle and any stencil or test that names manifest/.


## From `run-parentage`'s PR-Gate (2026-10-02): retire `seed.Parent`

`shedrun.Seed.Parent` is no longer written; parent comes from the pair's origin record, and the field is read only as a fallback for runs seeded before `aa67fd1d7`.
Remove the field and the fallback once no such run remains.


