```yaml
lens: fix-claims
outcome: reported
```

Checked the claims that the fix added or rewrote against HEAD.

1. **MEDIUM. D3's "so the assistant never outlasts the chair's wait" claims more than the rule secures.** The changed D3 text refuses only an `assist_reviewer_timeout_min` that is not below `review_timeout_min`, so 239 against 240 passes. D1's Bound, as changed, says the chair's wait also includes the assistant's start, with its start retries, plus one await cap, and that these "fall in the margin between the two". That holds at the defaults, a 180-minute margin from `review_timeout_min: 240` (`internal/loomengine/template.yaml:24`) less the 60-minute default. A margin of one minute is shorter than an await cap, though, so the chair can still time out inside its wait. D3's "never" and D1's "fall in the margin" hold only when the margin covers the start and one await cap. The validation rule as stated does not secure that.

2. **LOW. D12's "A render that names `claude-file` renders `codex-file` with it" is written as present-tense fact, but HEAD does the opposite.** `Render` skips every target that `req.Only` does not list (`internal/heddleengine/render.go:96`). The record states the behaviour as decided, but no Scope line or D12 acceptance criterion says the `Only` filter changes. A reader can take the sentence as a fact about the current code.

The other changed claims hold:
- The fan record is written once, before the attempt loop (`internal/shedadapters/burlerreview.go:229-233`).
- `fileText` returns false when no entry names the target (`internal/heddleengine/render.go:250-251`).
- The TUI rejects `--ignore-rules` (spike report, line 291).
- `PATTERN-lym-temp-root` exists (`pattern/PATTERN-lym-temp-root.md`).
- The 180-minute default margin is correct.
- The reviewer's source claims about Codex `exec_policy.rs` and `storage.rs` were taken over as cited. I could not check them from this worktree.

Tool calls:
- grep of the loom template, the PATTERN files, the spike report and shuttleengine's `doc.go`: my context lacked the timeout default, whether the PATTERN file exists, the spike's `--ignore-rules` text, and how a Prepare failure maps to a start.
- grep of shuttleengine's `doc.go` and `NotStarted`: my context lacked how a refused start is reported, to check D5's "marker reading `not started`" claim.
