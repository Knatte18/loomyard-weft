```yaml
lens: walkthrough
outcome: reported
```

**Decisions — case — purpose met?**

- **D1** — stored record for slug `x`, then a render. Default prefix `design-`: met. With `design_prefix: ""` (or `decision`, `de`, `decision-f`): **not met** — the template allows any prefix via `${env:LYM_DESIGN_PREFIX:-design-}`, and `renderDesigns` names a design file `designPrefix+slug+".md"`, so an entry slugged `decision-x` renders `decision-x.md` over entry `x`'s record and puts it in the manifest.
- **D2** — store on a run-held entry (status running): `checkRunLocks` passes (`scopeChanged` lacks the flag), `stampDates` stamps no revised date, save/render/sync. Repeat with same bytes → `noWrite`. Met.
- **D3** — `get` output with `discussed: true` fed back to upsert → no-op; new entry with `discussed: true` → refused. Met.
- **D4** — prune of done entry `foo`: `afterSave` removes `decision-foo.md`, sync commits both; on replay with origin moved, the commit is set aside as all-ended, same as for `board.json` today; file/entry consistent. Met.
- **D5** — local store commit (board.json flag + `decision-foo.md`) while origin moved: all paths claimed, `CommitRuns` names `foo`, record by design-file rule, flag merged by field. Met.
- **D6** — judged CONVERGED on Discussion-Bouncer: `writeCarryOver` → `approveAndCommit` (Approve = store, then Commit = `commitDiscussion` staging `_lym/discussion` unchanged). Stored bytes == committed bytes. Slug: `SetRunStatus` gets `ResolveRunID` = worktree name. Met.
- **D7** — tapestry: DiscussionWrite's `on_done` = `Discussion-Bouncer`; Plan-Bouncer has `approve_seam: plan`, Code-Bouncer none; guard holds. Met.
- **D8** — `decision add` at Plan-Write copies committed record. Met. Case 2: a later Plan-Review / Code-Review settle runs `writeCarryOver`, which writes into the decision record and commits it; no `decision add` follows, so the board copy stays stale. **Not met** (see F2).
- **D9** — `store` at Webster copies committed record at HEAD. Met.
- **D10** — `--record` beside `--body` refused; on discussed entry prints file. Met.

**Findings**

F1 — MEDIUM — design — decision-record.md:32 (D1)
Summary: refusing only a `design_prefix` equal to the record prefix does not stop a render from writing over a record file.
Concrete: `design_prefix: ""` (allowed by template's env default) plus an entry slugged `decision-x` with a body renders `decision-x.md`, overwriting entry `x`'s record and adding it to the render manifest; prefixes like `de`, `decision`, `decision-f` collide likewise. The purpose "the render never owns a record file" fails.
Evidence: internal/boardengine/render.go:795 `fmt.Sprintf("%s%s.md", designPrefix, e.Slug)`; internal/boardengine/template.yaml:2 `design_prefix: ${env:LYM_DESIGN_PREFIX:-design-}`; internal/boardengine/config.go has no prefix validation.
Fix: refuse any prefix for which `prefix+slug` can equal `decision-<slug>.md` — i.e. any prefix that is itself a prefix of `decision-` or that starts with `decision-`.

F2 — LOW — consistency — decision-record.md:103 (D8) and Goal (decision-record.md:4)
Summary: the board copy goes stale after a later segment's carry-over, while the record implies it stays current: Plan-Review / Code-Review settles file their carry-over into the decision record and commit it, but neither the approve seam nor `decision add` runs then, so the stored copy lacks those blocks until the next `decision add` or a hand `decision store`. D8's "carry-over blocks later review segments wrote into the record reach the board copy too" and the Goal's "keeps the stored copy current" hold only if a `decision add` comes later.
Evidence: internal/loomcli/wiring.go CarryOver wiring comment ("files each review segment's still-open findings in the decision record and commits that record alone"); internal/shedadapters/bouncer.go:1164 `writeCarryOver` on every converged settle of any segment.
Fix: either state the board copy is refreshed only at approval and on `decision add`, or store on later segments' carry-over too.

Tool calls:
1. internal/loomcli/wiring.go:336-400 & 845-875 (commitDiscussion, ApprovePlan, CarryOver wiring) — to confirm the commit seam does not transform the record and what writeCarryOver writes; my context had only grep line numbers.
2. internal/shedrun/resolve.go ResolveRunID and template.yaml design_prefix default — to check the slug the approve seam would use and whether the prefix is free-form; neither was in context.
