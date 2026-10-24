```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: what each card's change forces outside the members it names.

- BLOCKING — Card 21 (tender-every-orch): the tenderengine test helper that fills the seams card 21 removes is not listed, so the package's tests stop compiling.
  Card 21 replaces `OrchStrand`, `OrchRevivePending`, `InjectOrchRevive` and `OrchWatcherLive` on `tenderengine.Deps` with `Orchs`.
  The test helper `internal/tenderengine#env.deps` fills all four, at `internal/tenderengine/pause_test.go:310`, `:311`, `:316` and `:341`.
  The card lists only `TestPause` and `TestRevive`.
  Fix: add `internal/tenderengine#env.deps` under card 21's `**Edit:**`.

- MEDIUM — Card 9 (D1, "Only a task is claimable"): `lyx board promote` of a project entry is left unguarded.
  `Store.Promote` (`internal/boardengine/store.go:700-706`) skips only `KindTask` and otherwise sets `Kind = KindTask`.
  After card 9, promoting a project either turns it into a task silently, or fails on card 9's new "order/beam only on a project" refusal with an unrelated message.
  No card lists `internal/boardengine#Store.Promote` or a row for it.
  Fix: card 9 (or 10) edits `Store.Promote` to refuse a project, naming `lyx beam promote`/`create` as the way, and adds a `TestPromote` row.

- LOW — Several cards put a doc under `**Extends:**` while their change makes its existing text wrong, so the doc needs `**Edit:**` (rubric "`Extends` on a ref the card rewrites"):
  - cards 9 and 10, `internal/boardengine/doc.go`: "every entry in it is a task or a note", "Only a task is claimable", the README section list and Tasks layering, and Rename's "every other entry's depends_on";
  - card 7, `internal/fabricengine/doc.go:1129`: "`RegenerateHubWorkspace` writes … the one VS Code workspace file of a hub";
  - card 19, `docs/overview.md:407`: the orch row's "prime only";
  - card 23, `docs/overview.md:427` and `:432`: cleanup and adopt "prime only", and `internal/adoptcli/doc.go:6`: "It runs from the hub's prime only";
  - card 21, `internal/tenderengine/doc.go`: "never touches the orch" / "the orch" read as the prime's.

- LOW — Card 19 (orch-in-beam): the doc comment of `internal/orchcli#resumeContextCommand` (`internal/orchcli/resumecontext.go:19`) becomes wrong.
  It says the hook changes directory to "the prime's anchor", but `internal/orchcli/start.go:109` passes `c.location.AnchorPath()`, which is the beam's anchor once card 19 admits a beam.
  The orchcli package doc's "session-start hook" section says the same, and card 19 lists neither.
  The code is right; only the comment and the doc need the change.

Tool calls made:
1. `lyx quarry resolve` over the cards' unverified glyphs (test, helper and wiring members), to check they exist; the context held only names.
2. Re-ran that resolve through `jq` filtered to non-found answers (it printed none), and grepped `drive.go` and `cancelcmd.go` for the functions that enclose the `NotifyPrime` and `orchStrandRecorded` calls, to confirm cards 25 and 15 cover them.
3. Grep for the resume-context hook, tender seam fillers, skip-text assertions, the TestMain rule, skill docs and `Add` calls with a variable options value, since the pre-read did not cover these.
4. Read the hook builder, the tender pause test's seam usage and the hermetic-env rule, to confirm the two candidates.
5. Grep for the hook's caller, boardcli's text listing, reportengine's recipe set, recipe-row readers and `KindTask` users outside boardengine, for forced edits from the new kind and recipe.
6. Found the function enclosing `pause_test.go:341` and checked tests and docs for the removed tender seams, to name the missing member.
