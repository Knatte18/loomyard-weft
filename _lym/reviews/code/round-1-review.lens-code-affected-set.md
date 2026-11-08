```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens, covering what the removal of the `[slack] ` body prefix forces outside the lines the diff touches.

**Per change, the set found:**

1. **`forwardPrefix` removed from `inbound.go`, and `forward` posts the bare text.**
   - No other reference to `forwardPrefix` remains. A repo-wide grep for `[slack]` and `forwardPrefix`, excluding `.git`, `_lym`, `.lym` and `docs/research`, returns only the `[slack presence]` and `[slack relay]` strings.
   - No production code keys on a `[slack]` body prefix. The `HasPrefix` hits are all in tests and match `[slack relay]`.
   - The creel body limit check in `inbound.go` now uses the bare length. The `AtTheCap` and `OverTheCap` rows in `inbound_test.go` were updated to match (4096 and 4097). Covered.
   - Every forwarding assertion in `inbound_test.go` and the `forwarded` helper in `relay_test.go` were updated. Covered.

2. **`role-slack.md` now treats any other mail from `<shortname>:SLACK` as the operator's words.** This holds only if every non-operator body from SLACK carries a prefix. Every `Post` site in `slackengine` does:
   - `holder.go:520`, `keeper.go:498`, `relay.go:323`, `outbound.go:374` and `inbound.go:611` all use `relayNoticePrefix`.
   - `inbound.go:201` uses `presenceNoticePrefix`.
   - `inbound.go:683` is the operator forward.

   `dobbydeps.go:277` and `slackcli/wiring.go:202` only address mail to SLACK. They do not send from it. The old `[slack]` rule is fully gone from the role file. Covered.

3. **`docs/overview.md` and `internal/slackengine/doc.go` reworded.** No other `[slack]` prefix description remains in `contracts/`, `docs/slack-app.md`, `slackcli`, `outbound.go` or `beamcli`. Covered.

4. **PATTERN and commit message.**
   - The diff touches no `PATTERN.md` or `pattern/` file, and the constraints section declares no changes, so no declared entry is missing.
   - The commit has no `Pattern-Departure:` trailer, and none is needed.
   - No stencil changed, so the stencil-wording check does not apply.

**Findings:** none.

**Tool calls (1):**
- One grep for `[slack]`, prefix and "opens" wording across `contracts`, `docs`, `slackcli`, `beamcli/dobbydeps.go` and `outbound.go`. My context lacked docs beyond the diff, so I used it to look for leftover descriptions of the prefix.
