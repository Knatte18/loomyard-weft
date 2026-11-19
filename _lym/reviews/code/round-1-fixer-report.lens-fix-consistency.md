```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F14 change set.

**Findings**

1. **LOW** — `internal/loomcli/parentreview.go:333`: the `parent-review` help text says a "Discussion-Review or Code-Review round's ask to review beside the reviewer". The Code-Review ask now goes out before the Bouncer judges, and the judge waits for it, so "beside the reviewer" is stale. The F12 text fix left this twin as it was.
   - Evidence: `parentseat.go` now says the Code-Review ask is sent by the Bouncer's judge call, not at round start.

2. **LOW** — `contracts/orchrole/orch-role.md:262` and the first sentence of the drift section: the text still says the ask is sent "once per judge spawn". The F2 fix means a rewritten judgement, or an open ask on other rulings, sends a new ask in the same spawn.
   - The edited line at 265 now says the judge "asks you again", so the section contradicts itself.
   - `driftengine/judgeparent.go` records the digest per ask and re-asks.

3. **LOW** — `internal/driftengine/template.yaml:11` (`judge_ask_parent` comment): it still says the judge waits "for the parent to agree or change rulings before they are stored". It does not mention the re-ask on rewritten rulings.
   - I did not check this against the edited `judge_parent_timeout_min` line, only the stale wording.

**Checked, no finding**
- No `AskEachRound`, `roundParentAsk`, `settleRoundParentAsk` or `heldParentAsk` remains outside the rewritten files.
- `docs/overview.md`, the `code_review_*` template comments, `config.go`, `shedadapters/doc.go`, `loomcli/doc.go` and `driftengine/doc.go` agree with the new flow.
- `bouncer.go` `ParentAsk` docs agree with the `parentseat.go` field docs.

**Further tool calls (3)**
- Combined grep for stale names and phrases across `internal`, `contracts`, `docs` and `pattern`: my context lacked a repo-wide search.
- `sed` of `loomcli/parentreview.go:326-345`, the help text, `orch-role.md` line 262 and a `driftengine/doc.go` span: those spans were never read in changed form.
- `grep`/`sed` of `loomengine/template.yaml` lines matching "reviewer keeps"/"the chair": my context lacked the other template keys' comments.
