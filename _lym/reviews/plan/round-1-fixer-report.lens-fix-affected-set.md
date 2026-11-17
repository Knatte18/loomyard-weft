```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran the searches for F4 (escalation-cause enumerations) and F5 (validate-review texts), which are the only set findings.

List: F1, F2, F3, F6, F7, F8 and F9 were left unchecked because none of them rests on a set. F4 and F5 were checked.

- **LOW**, about Card 8's Edit list and its Intent sentence "The `validateReviewVerb` comment no longer says the check reads only the file it is given". F5's set has a member the fix leaves out: the `resolvePersistentPreRun` doc comment still says the verb "reads one told path and needs no hub, mode or engine wiring". After Card 8 the verb also reads the run directory and the decision record, so that comment becomes false the same way the `validateReviewVerb` comment does. Card 8 neither lists `internal/burlercli#burlerCLI.resolvePersistentPreRun` under Edit nor names the comment in its Intent. Evidence: `internal/burlercli/cli.go:65`.
- **LOW**, about Card 10's Intent sentence "`EscalationStalemate` joins `valid`; the doc comments ... list five". The const group's comment "The four legal EscalationCause spellings" (`internal/shedadapters/escalation.go:22`) is the comment over the block that `EscalationStalemate` joins. The card names no glyph that owns that comment, and the fix's added sentence names only `ErrEscalationMalformed` and the file header. Whether "the doc comments" covers it is ambiguous. The other members are covered: `valid` (`:30`), `readEscalation` (`:79`), `circling.go` (Card 11) and `loomcli/doc.go:577` (Card 11).

Tool calls:
- Grepped for escalation-cause enumerations to rerun F4's search; my context held only the sites the review named.
- Grepped for validate-review texts to rerun F5's search, for the same reason.
- Read `cli.go:58-70`, `escalation.go:18-24` and `escalation.go:74-82` to identify which symbols own the unlisted hits.
