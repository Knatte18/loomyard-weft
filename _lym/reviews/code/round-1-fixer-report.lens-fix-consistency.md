```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale siblings through the consistency lens.

Findings:

1. **LOW** — `internal/gatecli/doc.go:316`, "The claim is an exclusive create of `process.json`". The F5 fix makes the claim a temporary file linked to `process.json`.
   - Evidence: `claim` now uses `os.CreateTemp` and `os.Link`.
   - The sentence is no longer accurate. It should say the claim links a complete temporary file to `process.json`, so the process file never appears empty.

2. **LOW** — `internal/gatecli/doc.go:321`, "cancels the call once the lease has passed". The F3 and F4 fixes changed the rule.
   - The lease loop now cancels only on a lease read as lapsed, and an unreadable lease is retried.
   - The runner marks abandoned only if the cancel came before the call returned (F4).
   - Line 322, "a passed lease writes code 1 and the abandoned mark", is therefore too broad. Adding the two bounds would bring the doc in line.

3. **LOW** — `internal/gatecli/doc.go:339–346` and the retention description. The F2 and F6 behaviours are not documented.
   - F2: a run whose output copy failed is left undelivered. Line 346 says `delivered` is created after the output is printed.
   - F6: the front now prunes run directories older than 14 days whose runner is not live, and nothing in `doc.go` mentions it.

4. **LOW** — `internal/gatecli/doc.go:322`, the abandoned line. The F7 fix names the request's verb in `abandonedLine` and in the going and busy messages. The doc names only `gate test` and does not say the line follows the verb. It says "an abandoned line", which is still true, so this is minor.

Checked and consistent:
- Every code and test use of `replayedBusyLine`, `abandonedLine` and `answerRunGoing` carries the new signatures. A repo-wide grep finds no remaining old literal.
- `docs/overview.md` F9 matches the template keys `keep_warm_cap: 250s` and `keep_warm_cap_roles`.

Calls made:
1. A grep of `doc.go` for claim, lease, delivered and pruning wording, because my context did not hold `doc.go`.
2. A grep for the old `gate test: the run…` and `no gate slot…` literals, to confirm no twin of the F7 change was missed.
