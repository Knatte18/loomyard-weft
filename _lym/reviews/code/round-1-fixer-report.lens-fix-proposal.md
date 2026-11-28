```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer proposals the fixer carried out (F1–F6).

**1. BLOCKING: `_lym/landing/summary.md`, `## Merge danger` (F2)**
The rewritten section no longer passes the change-description check. The proposal asked for a conditional door and a multi-part blast radius, and the format allows neither.
- The door is now `one-way once a layer holds `unattended`; two-way while no layer sets it, …`. `mergeDangerFindings` in `internal/summaryparser/description.go` accepts only a value equal to `one-way` or `two-way`. `labelledValue` trims only spaces, backticks and a final period.
- The blast radius is now a long phrase. The same function refuses any value where `len(strings.Fields(radius)) != 1`.
- `contracts/specs/loom-final-summary-spec.md` says that `lym loom validate-description` and every gate that checks a description refuse both.
- A sound form is `**Door:** one-way` and `**Blast radius:** beam` (or `config`), with the conditions in separate prose lines below the two labelled lines. Then run `lym loom validate-description`.

**2. LOW: the added `unattended` row in `internal/loomcli/autonomy_integration_test.go` (F4)**
The proposal closes only one of the three sites the finding names. Changing `.On()` to a comparison with `AutonomyOn` at `internal/loomcli/pawlwiring.go:154` or `internal/slackcli/wiring.go:131` still passes every test. The review's own `fix` text says so. The fixer-report should say this rather than record F4 as wholly closed.

**3. LOW: `tellRefusal` posting unlocked (F6), and the summary's "preceded by" sentence**
With two stamps of one run at once, the second sees the pending entry, skips the post and returns. Its mail can then arrive before the refusal mail. If the first post fails, that mail goes out with no explanation until the next refused read.
So the summary's "the first stamped mail is preceded by one mail to the dobby's orch" holds only for sequential stamps. I did not establish whether the dobby's three posting seams ever run concurrently.

**4. LOW: episode ends only on a successful read (F5)**
No test fails if `if read {` in `tellRefusal` goes back to ending the episode on any nil refusal. The new later-reads loop uses only a good read and a refused one, so the behaviour F5 asked for is unpinned. The `hubgeom` tests pin only the returned `read` flag.

Nothing found against the F1 proposal, or against the F3 episode-end rows as changed to use watch lines.

Tool calls beyond context:
1. Grep for the `Door` and `Blast radius` labels, to find whether anything parses the section; no parser was in context.
2. Read `internal/summaryparser/description.go` and `contracts/specs/loom-final-summary-spec.md`, to get the accepted values; neither was in context.
