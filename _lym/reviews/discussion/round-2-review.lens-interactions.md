```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens on `decision-record.md`. I check every pair of decisions that act on the same state, and the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D2 (ask before the hold) × D4 step 1 (tip read after the hold): held.
- D2 / D3 hold (`lym tender pause <slug>`) × D4 step 2 failure lift (`lym tender resume <slug>`): held.
- D2 slug pause × D4 step 3 cancel: held. Only a hub pause refuses (`cancel.go:299-304`). The cancel exempts itself from the gate (`:306`) and lifts slug entries on completion (`:572`).
- D4 step 3 partial/refusal rule × D7 lift × D2 hold × D4 step 3 tag name: broken (I1).
- D4 step 3 code-tag wait × D4 step 4 `--record-file` × D4 step 5 records-branch wait: broken (I2).
- D4 step 2 parent check × D4 step 4 `project` write, across step 3's wait: broken, minor (I3).
- D4 step 1 recorded tip × D4 step 3 rerun after a partial finish: held. A handed-off remote failure reaches the orch only as a notice, and the notice's tag is the local tip's (`remoteteardown.go:186-188`).
- D2 note in the worktree × D4 step 3 `--remove`: held, since D2 copies the note to `.scratch/` first.
- D6 read through the archive tag × the owner Constraint ("needs nothing local … once D4 step 3 has confirmed the tag on origin"): broken (I4).
- D4 step 5 prune × D5 same-slug re-run: held while `stale-records-ref-blocks-rerun` is open. The cross-orch release names the prune.
- D4 step 3 end-only × D1 bound (the board edit is legal only after draft): held, since D4 re-runs until the cancel completes.
- D7 exclusions × D4 step 2 candidate parents: held.
- D8 glossary × D3 (the orch as author): held after F7.
- Acceptance criteria D1–D8 × their decisions: held, except that D4's bullet carries I1 and I2 by mirroring them.
- Constraints (owner machine, draft, `--remove` tags) × D4: held, except the owner Constraint × D6 (I4).
- Open risks line × D4 step 5: held. It is a pointer to the step.

**Findings**

I1 — MEDIUM, design, `_lym/discussion/decision-record.md:90-94` (D4 step 3) and `:141` (D7)
Summary: the partial-finish rule sorts the hub-pause refusal as a partial finish. The rule says an error whose way forward names the cancel to run again is a partial finish. The hub-pause refusal names "run `lym tender resume`, then … again", so the orch runs the hub-wide `lym tender resume`. That command lifts every pause entry, D2's slug hold included. The run then resumes and commits before the second cancel. The cancel archives a tip other than the one recorded in step 1. Step 3's `ls-remote` waits for a tag named from the recorded tip, and that tag never appears; no failure notice arrives, since the push succeeded. The owned-elsewhere refusal has the same flaw: it names both `lym adopt` and `lym batten cancel <slug> on <owner>`, so it fits both cases.
Evidence: `internal/battencli/cancel.go:303-304` (hub-pause wording), `:715` (owned-elsewhere wording); `internal/tendercli/cli.go:194` (resume with no slug lifts every entry).
Fix: classify by whether the cancel has acted, not by whether the message names the cancel. A hub-pause refusal and an owned-elsewhere refusal are refusals: D7 applies, and the orch never runs a hub-wide resume. Alternatively, have the orch re-read the tip and re-derive the tag name after any resume.

I2 — MEDIUM, design, `_lym/discussion/decision-record.md:96-110` (D4 steps 3–5)
Summary: step 4's `lym board upsert --record-file` for a discussed entry is refused while the slug's records branch is still live on origin. Step 3 waits only for the code archive tag. The wait for the records branch to disappear (`ls-remote` showing no records branch) sits in step 5, after step 4. The teardown child deletes that branch separately from the code tag push, so the procedure as written meets a refusal at step 4.
Evidence: `internal/boardengine/recordupsert.go:38-46` (a live weft branch refuses, with the way forward "wait for the pair's teardown to finish"); `internal/fabricengine/remoteleftover.go:86-93`, `:105-113` (liveness covers the origin branch until an archive tag covers it).
Fix: move step 5's wait for the records branch into step 3, or at least before the `--record-file` branch of step 4. Have step 4 take the refusal's way forward.

I3 — LOW, design, `_lym/discussion/decision-record.md:83-88` (D4 step 2) versus `:106`
Summary: the parent check runs before the cancel and the tag wait. The `project` write comes after both. In between, a beam can move past its hold-free rows or be flagged ready, and then the upsert is refused after the cancel can no longer be undone. D4 gives no path for that case: the entry stays a draft under its old parent.
Evidence: `internal/boardengine/project.go:97-100`; `internal/boardengine/runlock.go:44`, `:56-62`.
Fix: if step 4's upsert refuses the project, keep the entry a draft with the note written in, and tell the operator. Or repeat the check right before step 4.

I4 — MEDIUM, design, `_lym/discussion/decision-record.md:166-167` (owner Constraint) × `:129-130` (D6)
Summary: the Constraint says a fresh run on the new parent's owner machine needs nothing local once the tag is on origin. D6 has that run read `<merge base>..<archive tag>`, but no step fetches the tag. With default refspecs, git's tag auto-follow does not bring an archive tag whose commits sit on a deleted branch. On another machine the diff therefore fails, and the record does not say who fetches the tag or how. This is a tool fact the record leaves unpinned.
Evidence: `pattern/PATTERN-single-repo-hub.md` ("A code store keeps git's default refspecs"); `internal/fabricengine/remoteleftover.go:149`, `:183` (fabric fetches archive tags only by explicit refs).
Fix: have the move-note section in D4 step 4 tell the fresh run to fetch the tag explicitly, for example `git fetch origin refs/tags/archive/<slug>/<12hex>:refs/tags/archive/<slug>/<12hex>`, before reading the diff. Pin that fetch.

Outside this lens: D4 step 2 says to check with `lym board get` that `lym beam ready` has not run, but the ready flag is a record in the beam worktree's `_lym/landing`, not on the board (`internal/beamcli/cli.go:917-919`).

Tool calls: none. The inherited context covered every pair.
