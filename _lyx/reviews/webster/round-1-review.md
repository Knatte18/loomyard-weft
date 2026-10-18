---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/orchcli/paths.go:16
    summary: orchDirName comment says the directory is beneath the anchor's .lyx directory, but rulesRel now also joins it under _lyx
    fix: In internal/orchcli/paths.go change the orchDirName doc comment to say it is the module's own subdirectory name beneath the anchor's .lyx directory and beneath _lyx, where the rules files live.
    origin: handler
---

### [NIT] orchDirName comment no longer covers its second use

**Location:** internal/orchcli/paths.go:16
**Issue:** `orchDirName` is documented as the module's subdirectory beneath the anchor's `.lyx` directory.
The commit now also joins it in `rulesRel()` under `_lyx`, so the comment describes only one of its two uses.

## Rejected

- Affected-set fork: `Command`'s `Long` might list verbs and miss `rules`. The `Long` text in `internal/orchcli/cli.go` lists no verbs, so there is nothing to update.
- Tests fork: `TestWatcher_NoteReplyGate`'s new subtest does not assert the negative case "any other directory leaves the gate closed". The positive case already fails on a wrong root, and the fork itself judged it non-blocking.
- Pattern-fit fork: the `noHubRules` and `noOwnRules` comments call the `%s` placeholder a "verb". That is the standard `fmt` term for a format directive, not a finding.
- Correctness fork: no findings; every branch traced matched the cards.

## Notes

- Review range: no `_lyx/loom/rework` round exists, so the range is the merge base with `main` (`ff6b3e8c1`) to `HEAD`, the three card commits.
- The commits carry no `Pattern-Departure:` trailer and the diff needs none.
- The constraints section declares no PATTERN changes, and the diff touches neither `PATTERN.md` nor `pattern/`.
- No `ORCHESTRATOR.md`, `RepoRulesPath` or `repository_rules` reference remains outside the card's exclusions.
- Focus item 1: `lyx orch rules` runs under the unchanged `orch` group `PersistentPreRunE`, so `refuseNonPrime` precedes the engine config loads and the verb body.
  `TestWriteHubRules` has a row for each failure branch of `rulesBolt`, and `TestOrchIntegration_RulesCommitsOnBolt` covers only the real commit and push.
- Focus item 2: `readRules` reads the hub-wide file before the own file, and both paths join `rulesRel()` once.
- Focus item 5: the added tests in `prompt_test.go`, `start_test.go` and `watcher_test.go` each pin behaviour no existing test covered, and none asserts static stencil prose.
- Card 4 files a board note, not repository content, so the diff cannot show it and it is not judged here.
