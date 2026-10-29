```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: I checked what each card's change forces outside the members it names. Read-only; no files changed.

**Findings**

1. LOW. Card 4 (pr-review gate key): `internal/gateslot/doc.go` names the gate keys lym reads, and the card leaves it stale. The card lists the file under Uses only, but the doc's key list at `internal/gateslot/doc.go:61` and its row-to-key table at `internal/gateslot/doc.go:105` both end at `drift-verify`. Once `goReadGates` (`internal/gateslot/tiers.go:87`) gains `pr-review`, both are wrong. Fix: move `internal/gateslot/doc.go` to Edit (or Extends), add `pr-review` to the diff-scoped keys at line 61, and add a `loom, darn, pawl  PR-Review  pr-review  diff` row to the table. The example gates block at `internal/gatecli/doc.go:290-301` is a sample hub config, so it is optional.

2. LOW. Card 7 (new package `internal/pullrequest`): no card adds the new module to the module table in `docs/overview.md`. Its siblings `internal/hubissues/` and `internal/githubclient/` have lines there (`docs/overview.md:230`, `docs/overview.md:302`). CLAUDE.md requires `docs/overview.md` to change in the same commit when the module table changes. Fix: card 7 lists `docs/overview.md` under Extends with an `internal/pullrequest/` line.

3. LOW. Card 11 (the stub goes): two file header comments still call it a stub, and no card names them:
   - `internal/landingshed/prreview.go:1` says "implements the PR-Review stub".
   - `internal/shedrecipe/entries_prreview.go:1` says "builds the PR-Review stub".

   Card 11 lists only members of those files, and a file path can't sit beside its members (`redundant-file-target`). Fix: the Intent says both header comments are rewritten.

4. LOW. Card 11 (pawl recipe gains PR-Review): the header of `internal/loomrecipe/pawlrecipe_test.go:3` says "the pawl recipe's nine rows". With the new row there are ten, and the card only edits `internal/loomrecipe#wantPawlRows`. The same header lists "the absence of a reroute edge on Finalize" among the covered edges, and the `Beam-Describe` row's next row changes at `internal/loomrecipe/pawlrecipe_test.go:45`. Fix: the Intent names the header comment's row count.

5. LOW. Card 14 (`internal/landingshed#PullRequestDeps` is nested in `Deps`): the plan may miss a seam marker on `internal/loomcli#TestLandingDeps_EveryFieldPopulated`. That test carries `//lym:seam internal/landingshed#Deps` (`internal/loomcli/landingdeps_test.go:42`). Card 11 adds `//lym:seam internal/landingshed#PullRequestDeps` to the three envkit tests, but no card adds it to this test. Card 14 edits the test only to list the unfilled `SessionLive` and `RegisterSession`. If the test reflects into nested struct fields, as it must to see those two, it needs the marker too, or `TestSeamMarkers_ReflectiveTestsAreMarked` reports it. Fix: card 14's Intent adds the marker.

6. LOW. Card 9 (`lym loom pr-continue` as a role-audience verb): `clihelp.AudienceRole` is defined as a command "named by that role's prompt" (`internal/clihelp/annotations.go:25`). The stencil that names it arrives only in card 10. I found no test that enforces this, so it is not blocking. The ordering is still worth a sentence: card 9's Intent should say card 10's stencil is the prompt that names the verb.

**Calls I made beyond the inherited context**
- grep for `pr_review`, `NewPRReview`, the `buildSettings`/`buildDenyNotice` callers, `childAtBeamAwait`, stale "no pull request" wording and the gate-key lists. My context lacked the full set of their references.
- `lym quarry resolve` on the cards' existing glyphs, plus a grep of the claudeengine test callers. I needed to confirm the glyphs exist and that the re-sign's test callers are covered.
- grep for `AudienceRole`, the runproc role sets, agent role names, the pawl recipe test edges, beam.yaml key lists, loom verb lists and the overview module table. These are registries and lists the new names must join.
- Read of `internal/gatecli/doc.go` (the gates example), the board skill fields, `segmentcolor.Landing`, the pawl test row count and the `verifyGate` fields. I needed to check those doc lists and that the seams cards 13 and 14 cite exist.
- grep for `AudienceRole` tests and the function holding loom's `AddCommand`. I needed to confirm card 9's `Command` glyph and whether a role verb must be named by a prompt.
- grep for a committed CLI index or skill listing loom verbs. I needed to check whether card 9's verb forces an index edit; I found none.
