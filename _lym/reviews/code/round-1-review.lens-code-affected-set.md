```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — what the diff's changes force outside the lines they touch.

**Per change, the set found**

1. `Strand.PaneMail` and `hasHookDelivery` removed.
   - A repo-wide grep finds no leftover reference to either symbol.
   - `creelfake`, `deliver_test.go`, `delivery_test.go`, `deliver.go` and `seam.go` are all updated.
   - `shuttleengines` stays imported in `delivery.go` (`EnvScrub`, line 94) and elsewhere, so no import is left unused.
   - Covered.
2. Pane-mail wording in the creelengine docs.
   - Covered: `doc.go` lines 10, 81 and 150, the `Scheduler` comment at `deliver.go:33-36`, and `docs/overview.md` line 476.
   - Not covered: `deliver.go:200` still reads "types a notice or the reminder into a pane-mail strand", which is accurate but no longer says the pane-mail strand is any strand without `CreelHooks`.
3. The CLI help text in `internal/creelcli/cli.go`.
   - Not covered. See finding F1.
4. `docs/overview.md:312`, "reminders for a strand without hook delivery".
   - Still accurate after the change, so I raise no finding.
5. `confirmPasted` and the `submitted` flag.
   - Covered: the `confirmPasted` doc comment (`send.go:605`) and `shuttleengine/doc.go:275` are both updated.
   - Nothing in `docs/` or `pattern/` mentions this behaviour. `PATTERN-pane-text-injection.md` does not describe the confirmation rules.
   - Callers and tests: `checkLandingStartConfirmation` carries the new rows. I did not check other `confirmPasted` tests, which belong to the tests lens.
6. `smoke_tmux_test.go`.
   - Not covered. See finding F2.
7. PATTERN declared changes.
   - The rubric's constraints section declares none, and no `PATTERN.md` or `pattern/` entry mentions `PaneMail` or `hasHookDelivery`. Nothing is required.
8. Commit messages.
   - Neither commit carries a `Pattern-Departure:` trailer, and none is needed.
   - The commits say "Refs #897" and "Refs #899"; the summary claim that both are closed is the handler's to check.

**Findings**

F1 — MEDIUM, consistency. `internal/creelcli/cli.go:76-77` and `:315-317`
- Summary: the operator-facing `lym creel` and `lym creel post` help still says "A strand whose engine is unknown or has no hooks, a hand-added one included, gets a typed notice or reminder".
- Why it is wrong now: after D3 the rule is "every strand not marked `CreelHooks`", so an engine that has hooks but a session launched without them also gets the notice. This is the text the commit message and `docs/overview.md:476` were changed to correct, and the CLI help was missed.
- Evidence: `cli.go:76` and `cli.go:315`. The overview was updated in the same diff, so the help text is the one remaining statement of the old rule.
- Fix: reword both to "Any other strand, a hand-added one or a session launched without those hooks included, gets a typed notice or reminder…".

F2 — LOW, consistency. `internal/creelcli/smoke_tmux_test.go:76-78`
- Summary: the doc comment on `TestSmokeCreelFakeProviderPane` was corrupted.
- The diff replaced its first line, "TestSmokeCreelFakeProviderPane runs the real daemon on the real socket…", with a new sentence, but the old second line stayed. The comment now opens with "The strand carries…", twice, and no longer begins with the symbol name.
- The surviving line still says the provider tag `script` is one "no engine renders hook delivery for", which is no longer the reason the strand is a pane-mail strand.
- Evidence: `smoke_tmux_test.go:76-78`.
- Fix: restore the first line and make a single second line: "The strand carries the provider tag `script` and no `CreelHooks` mark, so it is a pane-mail strand."

F3 — LOW, consistency. `internal/creelengine/deliver.go:200`
- Summary: the `probe` doc says "a pane-mail strand" without the new definition. This is minor, since the `Scheduler` comment defines the term, so I rate it LOW.
- Fix: say "or types a notice or the reminder into any other strand".

Not covered by any search: whether `pattern/PATTERN-pane-text-injection.md` should state the submitted-landing-send rule. It does not describe the confirmation rules at all, so I raise no finding.

**Tool calls**
1. grep across `PATTERN.md`, `pattern/`, `docs/` and the CLI docs for the old pane-mail and hook-delivery wording; my context lacked the CLI help text and the pattern files.
2. Read `deliver.go` lines 195-235 and grepped `docs/` and `pattern/` for confirmation-rule mentions; my context lacked `probe`'s full comment and any pattern text on confirmation.
3. Read `cli.go` lines 72-80 and 312-320 and `docs/overview.md` line 312; my context lacked the exact help text.
