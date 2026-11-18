```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: for each change in the four cards, I checked the callers, tests, comments and docs it makes wrong that no card lists, and whether each listed glyph resolves.

No BLOCKING or MEDIUM findings. Every existing glyph the cards list resolves with `lym quarry resolve`. `fakeStrands` is the only implementer of `strandOps`. The Create handles' packages match: all `internal/loomengine` tests use `package loomengine`. Every doc that describes the runs strand replacing the checklist is on card 2 or card 3: `docs/overview.md` lines 424, 544, 687 and 821, `internal/orchcli/doc.go:13` and `internal/loomcli/doc.go:62`.

1. **LOW, card 3, test fake:** `internal/orchcli#fakeAdd` needs a new field that no card lists.
   - The Intent records the unless-name beside each entry of `addedUnless []fakeAdd`.
   - `fakeAdd` (`internal/orchcli/cli_test.go:31-34`) holds only `spec` and `guid`, so recording the unless-name forces a new field or a new type.
   - Fix: list `internal/orchcli#fakeAdd` under Edit, or declare a new record type as a `plan:` handle.
2. **LOW, card 3, D1:** the `ensureRunsStrand` doc comment turns false, and the Intent does not say to rewrite it.
   - `internal/orchcli/runs.go:49` says "it replaces a loom status strand instead of adding beside it".
   - `internal/orchcli/runs.go:51` says "It touches no other strand".
   - After card 3 the call re-adds the loom status strand. The member is listed, so only the Intent needs a line saying the comment is rewritten.
3. **LOW, card 1, D2:** the `LoomStatusStrandName` doc comment goes stale and no card lists the constant.
   - `internal/loomengine/driver.go:25-27` says the orch bring-up reads the role only "to take the session's overview window from that strand".
   - After card 3 the bring-up also uses it as the re-add's unless-name, and after card 2 `lym loom start` does too.
   - The constant is only on card 1's and card 3's `Uses:`, so no card edits the comment.
4. **LOW, card 2, test file headers:** two headers become wrong and no card lists them.
   - `internal/loomcli/sharedbootstrap_test.go:1-5` says no test there drives `ensureStatusStrand`, and that its branches are covered only through `resolveStatusStrandAction`. Card 2 adds `TestEnsureStatusStrandOn` to that same file.
   - `internal/loomcli/smoke_drivingsurface_test.go:6` says "ensureStatusStrand's branches are pinned at Tier 1 through resolveStatusStrandAction".
   - Card 2 lists the smoke test's function, so the file itself can't be listed (`redundant-file-target`). Fix: name both header rewrites in card 2's Intent.
5. **LOW, card 4, D3:** a file header and the command's `Short` text go stale and are not named.
   - `internal/battencli/overview.go:1-2` describes the window as "the dobby's lines … above the orch lines", but the frame gains a self-run section between them.
   - `overviewCmd`'s `Short` (`internal/battencli/overview.go:481`) reads "its dobby and its orchs".
   - The Intent names only the `Long` text. Fix: name the header and `Short` rewrites in the Intent; `overviewCmd` is already listed, and the file can't be listed beside its members.

Further tool calls:
- grep across `internal` and `cmd` for the deleted helpers, the role name, `strandOps` implementers and the test package clauses: my context had only the subagent's summary, not a full repo-wide list.
- read `smoke_drivingsurface_test.go:95-135`, the `loomengine` package doc, and `doc-go-unlisted` and `seam-test-unlisted` in `loom-plan-checks-spec.md`: to check card 2's smoke-test rewrite and whether any doc or seam-marker edit is forced.
- `lym quarry resolve` on the first batch of listed glyphs, plus a grep for `//lym:seam` markers and the smoke fixtures' location type: glyph validity can only come from quarry.
- `lym quarry resolve` on the remaining glyphs, filtered to failures, and the signatures of the overview test helpers: the first call's filter showed nothing useful.
- grep of `contracts`, `docs`, `pattern` and every `doc.go` for text about the checklist's window: to find doc edits beyond what my context held.
