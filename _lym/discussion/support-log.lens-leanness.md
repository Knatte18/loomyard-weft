```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the creel-agents decision record and support log, looking for content to cut or move.

**MEDIUM — Auto-mode assumptions.** Most bullets repeat decisions already in the record.
- The flag name `--expect-reply` is D9.
- The 15-minute interval and the two chases are D10.
- Trust widening to any orch strand is D2.
- Removing the `lym shuttle` verbs is D8.

Evidence: each bullet restates a Decision's text, so it states the same thing twice. Keep only the assumptions no Decision carries: that promotion settles the 2026-10-15 hold, and that `defer` is moot. The auto-pick narrative already lives in the support log's question ledger.

**LOW — Constraints.** Several bullets restate PATTERN entries generically and constrain nothing specific to this design:
- `PATTERN-test-speed` and `PATTERN-test-isolation`: "injected clock; hubforge".
- `PATTERN-stencil-ownership`.
- `PATTERN-tolerant-records`.

Evidence: the Plan writer gets PATTERN in full. Keep the bullets that carry a design fact: the creel no-config constant, the creelcli/orchcli cycle, and `PATTERN-daemon-wakeups` folded into the delivery loop. Cut or trim the rest.

**LOW — Constraints vs Notes for the plan writer.** The `creelcli`/`orchcli` import cycle is stated twice: once as the last Constraint and again as the first Notes bullet. Evidence: the same fact appears in both sections. Keep the Constraint and drop the Note.

**LOW — Open risks, second bullet.** "Whether Codex sessions can message each other is untested" says in its own words that this design does not need it, so it is no risk to this run. Evidence: the bullet's own second clause. Move it to the support log, or cut it; Scope/Out already covers Codex.

**LOW — Scope, "Docs" bullet.** It lists six doc files and overlaps the last acceptance criterion and CLAUDE.md's same-commit docs rule. Evidence: enumerating which doc files change is Plan-level detail. Keep the creelengine "Unbuilt" replacement, which the criterion pins, and cut the list.

**LOW — D5.** The bullets come close to a site enumeration that duplicates audit block 7, which D5's rationale already points at. Evidence: under the completeness test, the per-site verb choice is what the plan writer needs, and that should stay. Trim the bullets to the verb choice per role, without restating site contents such as "the batch-need ask and the every-batch-recorded message".

Nothing in the support log needs to move into the record. The Rejected alternatives section is correctly kept out of the record.

Tool calls made: none; my context already held both files and the rubric.
