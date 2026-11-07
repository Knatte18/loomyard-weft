```yaml
lens: code-affected-set
outcome: reported
```

**Affected-set list (per change, set found, covered or not)**

1. `templateFor` now takes a `*Registration` (was a `Shape`), and `hubAt` takes a template.
   - Set found: `CopyHub`, `SharedHub`, `CopyRegistered`, `copyTemplate` and `hubAt` in `template.go`, and `hub_test.go:579`.
   - Covered. The grep found no other caller in `internal`. `hub_test.go:579` is updated to `shapeRegistration`.

2. Failure messages changed: the copy messages now read `hubforge: copy of %q`, and the shape-outside-`Shapes` message gained "or a Registration".
   - Set found: the grep over `*.go` and `*.md` outside `_lym` found no test or doc asserting the old strings (`CopyHub: `, `hubforge: template %q: …`, the old outside-`Shapes` text). `TestSharedHub` and `TestCopyHub` assert only that a failure names the fixture.
   - Covered.

3. `commitRelocatedBinding` and the package doc now say "board's HEAD commit" and "history as long as the template's", where they said "the one commit".
   - Set found: the comment in `template.go`, `doc.go:46` and the summary.
   - Covered. `doc.go:46` carries no stale "one commit" wording.

4. Declared PATTERN change on `PATTERN-test-isolation`.
   - Set found: `PATTERN.md:55`, whose text equals the declared new text exactly, and the `pattern/PATTERN-test-isolation.md` diff.
   - Covered. The index line already carries the declared wording ("hub fixtures via `hubforge`"). The background diff adds the registered template and the build-function clause under "Hub fixtures", which matches the decision record's constraint.
   - The summary's "index line is unchanged" is consistent with that.

5. `doc.go` claims against the code.
   - Set found: `doc.go:22-58` against `template.go`.
   - Covered. The claims match the code: the refused calls, the skips, the tip check read from the bare, the per-key lock, failure marking, shared-key refusal, and the list of process-wide calls the TB cannot see.

6. Commit message and summary claims.
   - Set found: the commit `ec5cdfe1` message, `_lym/landing/summary.md`, and the diff.
   - Covered. The refused skips and "shapes build through registrations of their own" are both in the diff. No `Pattern-Departure:` trailer appears, and none is needed: the record's D3 pack step is correctly left to a later task, and the summary says so.

**Finding (one):**

- **LOW, consistency.** The `PATTERN-test-isolation` background states a rule that `template.go` and `doc.go` do not state in the same terms, and it widens the record.
  - Location: `pattern/PATTERN-test-isolation.md`, Hub fixtures section.
  - Evidence: the background says a build uses "no code whose result such a registration changes, such as a Bolt replay". The `Registration.Build` comment at `template.go:73-74` lists only the five process-wide calls and omits the "no code whose result a once-only registration changes" clause. `doc.go:52-58` also omits it.
  - Effect: the rule document carries the Bolt-replay clause, but the doc comment a build author reads does not. The decision record puts that clause in the bound.
  - Fix: add the clause to the `Registration.Build` comment.

No other finding. The `PATTERN-test-isolation` change is landed, and no stale caller, test or doc was found.

**Tool calls:**
1. `grep -n "test-isolation" PATTERN.md` plus a grep for the old failure-message strings across `*.go` and `*.md` outside `_lym`. Reason: my context lacked the current `PATTERN.md` index line and any other asserter of the changed messages.
