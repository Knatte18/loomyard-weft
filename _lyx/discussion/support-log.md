# Support log — kickstart-per-card

## Interview

Autonomous mode: no operator answers; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — exploration

Read the board entry, the related entries (`glyph-audit-tool`, `glyph-use-audit`, `enclose-kickstart`, `lighter-test-rules`, `review-inline-code`), `internal/websterengine/doc.go`'s kick-start pack section, the webster stencils, commits `da58615cd`, `1e39afc10` and `793c6a8ee`, the 2026-10-15 audit report, `planindex.Index.KickstartPack` and `lyx quarry resolve --help`.
Findings that shaped the design: the batch pack is already one `CardPack` per card; `KickstartPack` takes any card list; `plan:` handles bind only at record-batch through `Delta.BindHandles`; the fork runs in Master's session, which drives `lyx webster` verbs; `lyx quarry resolve` prints quarry's JSON, not the envelope.

### Turn 2 — batch 1: switch, overview, fetch

- Q1 switch shape. Recommended: a new bool beside `kickstart_pack_file`. Alternatives: an enum replacing it (cleaner, but rewrites a key landed today that hub layers may set). Picked the bool.
- Q2 default. Recommended: on, since the file arm has data and the per-card arm has none. Alternative: off, as `kickstart_pack_file` did. Picked on.
- Q3 overview. Recommended: the card files themselves. Alternative: a Go-rendered intent and glyph list per card in the prompt (duplicates the card file). Picked the card files.
- Q4 fetch. Recommended: a new `lyx webster card-pack` verb. Alternative: the fork runs `lyx quarry resolve` over the card's glyphs. Picked the verb.

### Turn 3 — batch 2: output, edges, trace

- Q5 output form. Recommended: plain markdown on stdout. Alternatives: envelope with an escaped markdown field; a file plus a path (one extra Read). Picked plain markdown.
- Q6 failure. Recommended: quarry failure degrades with exit 0; unknown card and unparsable plan refuse with a way forward; the prompt's fallback is the quarry or grep locate path.
- Q7 begin-batch under the per-card layout. Recommended: still resolve the batch for the empty rule and counts; write no pack file.
- Q8 trace. Recommended: `pack_layout=card` plus one `webster: card pack` line per call.
- Q9 closing section. Recommended: a `## Start here` section naming the first step; kept although the audit saw no parallelism gain from it in the file layout, since the per-card layout's first step (read every card file) is short and the end of the prompt is still what the fork acts on first.
  The per-card arm keeps a closing `## Start here` section that the audit found gave the file layout no first-turn gain, so a difference between arms is not attributable to the per-card fetch alone.
- Per-card reads may raise turns per card, since the fork reads in several smaller parallel turns instead of one; that is what the measurement is for (moved here from Open risks).

### Turn 4 — lens review

Six forks (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness); all returned.

- premises, BLOCKING: forks are refused every `lyx webster` command (`steerWebsterForkDeny` in `internal/shuttleengine/claudeengine/settings.go`, confirmed by the writer), so `lyx webster card-pack` could never run. Fixed: the verb is `lyx loom card-pack`, open to forks, `loomcli` already imports `websterengine`; D3 rationale and the no-denied-recovery constraint rewritten.
- premises, LOW: `plan:` handles bind only in record-batch; confirmed, no change. `renderLocate`/`renderReadRules` take two bools; added to Notes.
- goal-scope, MEDIUM: default on is a behaviour change the entry did not ask for. Fixed: kept on, named explicitly in Goal and Scope so the orch can veto it.
- goal-scope, LOW: second trace line not asked for. Fixed: Scope names it as an addition.
- goal-scope LOW and board-fit MEDIUM, merged: nobody carries the `card` layout into `glyph-audit-tool`. Fixed: Open risks names the orch's handoff and how the per-card prompt is told apart.
- goal-scope, LOW: `## Start here` confounds the arm. Fixed: stated in Open risks; D4 carries the rationale (also leanness 8).
- board-fit LOW and scenarios MEDIUM, merged: a run can mix layouts after deploy or a flip. Fixed: Open risks; acceptance criterion for a begin-batch re-run after a flip.
- board-fit, LOW: no conflict with `glyph-use-audit`'s read-set idea; no change.
- scenarios, MEDIUM: invalid inputs unspecified. Fixed: D3 refuses a non-numeric argument and a missing plan directory too, and renders any card of the plan, done or in another batch; criteria added.
- scenarios, LOW: `webster: card pack` has no batch key. Rejected: the verb reads no webster state by design, and the call sits in the fork transcript whose prompt names the batch.
- scenarios, LOW: fallback and sandbox-list text untested. Fixed: criterion added.
- scenarios, LOW: which tree the verb reads. Fixed: the worktree on disk, committed or not; criterion names an uncommitted edit.
- scenarios, LOW: re-fork of a partly done batch. Fixed: D2 says it reads every card file, as today.
- pattern-fit, MEDIUM: audience annotation unstated. Fixed: `role`, in D3 and Constraints.
- pattern-fit, LOW: refusal way forward unstated. Fixed: each refusal's way forward is the quarry or grep fallback.
- pattern-fit, LOW: engine must not import `planglyph`. Fixed: D3 says the CLI hands it the index.
- pattern-fit, LOW: `PATTERN-ref-shape-registry` citation unneeded. Fixed: dropped.
- leanness, LOW 1–3, 5–7: duplicated docs list, generic PATTERN restatements, duplicate `plan:` bullet, the fork tally, the rejected alternative in D3's rationale, the parenthetical in D4. Fixed: each cut or merged.
- leanness, LOW 4: the turns-per-card risk is deliberation. Fixed: moved to this log.

## Rejected alternatives

- Enum `kickstart_pack_layout` replacing `kickstart_pack_file`: changes a key landed today and any hub layer setting it, for no behaviour gain.
- Default off: the per-card arm would collect no data until a hub edit.
- Rendered overview in the prompt: duplicates the card files, against the pointer rule.
- Bare `lyx quarry resolve` per card: the fork must pick glyphs from the card and read quarry JSON, and the pack format splits across layouts.
- Resolving spans only at batch start and reusing them per card: line numbers go stale after an earlier card's edits, the board entry's reason for this task.
- Span code in the pack (variant 2): rejected by the operator.
- Cheapest form from the board's Idea (a new instruction "read card N's section of the batch pack before card N"): spans from batch start go stale by card N; the operator's decision requires fresh resolution.
- Binding `plan:` handles at card start: handles bind at record-batch, and the creating fork already holds the member it wrote.
- `lyx webster card-pack`: forks are refused every `lyx webster` command by the claudeengine hook.
- Narrowing that fork deny to exempt `card-pack`: weakens a deadlock guard and contradicts the fork prefix and Master template, where `lyx loom` needs no guard change.
- D7, teaching `tools/forkaudit` the per-card layout in this task: Merge-In runs after Webster in `contracts/recipes/loom-recipe.yaml`, so `glyph-audit-tool` cannot reach this run's Plan-Write, and the run lock refuses a `depends_on` change on a running entry.
  The card arm moves to a follow-up task (Scope Out); its marker row goes ahead of the `file` row because the per-card prompt also carries a `## Start here` heading, and without it every per-card fork is counted in the file arm, merging the two arms the measurement compares.

## Review rounds

### Parent review, round 1

- F1 (BLOCKING, decision): the audit tool's per-card support was out of scope, against `glyph-audit-tool`'s rule that a task adding a pack layout adds its `tools/forkaudit` marker row, and the per-card `## Start here` would class per-card forks as `file`. Fixed: Scope In names the work, new D7 adds a `card` marker row matching `lyx loom card-pack` ahead of the `file` row, the `card` trace value, and a span source from the `card-pack` tool results with the `webster: card pack` lines as cross-check; an acceptance criterion covers it; the Open risks bullet now names the dependency on `glyph-audit-tool` landing first. The span source is the tool results rather than the trace lines alone, since the trace lines carry counts, not spans.

### Burler review, round 2

- F1 (BLOCKING, design): D7 could not reach this run's Plan-Write and its `depends_on` hold existed only in prose. Fixed: D7, its Scope In bullet and its acceptance criterion removed; Scope Out names the card arm as a follow-up task depending on both `glyph-audit-tool` and this task; Open risks keeps only the interim `file` classification; Rejected alternatives records why.
- F2 (MEDIUM, consistency): D3's Bound understated what the verb writes. Fixed: the Bound names the logger's durable sink, the trace file and header, the Info and Warn lines and the exit sweep.
- F3 (LOW, consistency): D4's sandbox marker sat after `lyx quarry resolve` and deferred its text. Fixed: the marker sits after `lyx loom lint-pattern`, before `and`, with its fill pinned.

## Question ledger

- Q1 switch shape: new bool `kickstart_pack_per_card` — auto-pick.
- Q2 default: true — auto-pick.
- Q3 overview: the card files, read in one parallel turn — auto-pick.
- Q4 fetch: `lyx loom card-pack <card>`, moved from `lyx webster` after the premises lens found the fork deny — auto-pick.
- Q5 output: plain markdown on stdout on success, envelope on refusal — auto-pick.
- Q6 failure handling: degrade on quarry failure, refuse unknown card and unparsable plan — auto-pick.
- Q7 begin-batch keeps resolving the batch, writes no pack file — auto-pick.
- Q8 trace: `pack_layout=card` and `webster: card pack` per call — auto-pick.
- Q9 closing `## Start here` section in the per-card layout — auto-pick.
