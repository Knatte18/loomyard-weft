```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I traced what it forces outside the lines it touches.

**Affected sets, per change**

- **`launchOrch` no longer falls back to a fresh launch on every resume error.** The new `AbandonedSession` meaning is limited to "no transcript".
  - Covered: `orchcli/doc.go`, `orchengine/doc.go`, the `BeamUpReport` field doc, and the renamed row of `TestStart_StarterErrorFailsAndSavesNoState`.
  - Covered: the `beamcli/cli.go:433` `startEntry.AbandonedSession` passthrough and `beamcli/doc.go:309`. Both are still accurate.
  - Not affected: `fabriccli/doc.go:57`, `fabriccli/fabric.go:1128` and `pairteardown/teardown.go:75`. Their `abandoned_session` is the pair teardown's field.
  - Not affected: `claudeengine/command.go:424` and `claudeengine/doc.go:644`. Their "falls back to a fresh start" describes `OpenLaunch`'s `ResumeCmd`, not the orch launch.
  - No stencil names the fallback.
- **`CheckResume` returns `*SessionHeldError` and wraps `ErrNoTranscript`, and the refusal text changed.**
  - Claude is the only `SessionResumer` (`claudeengine/resume.go:33`), so no other engine is left uncovered.
  - No production code or test matches on the old message strings outside `resume.go`.
  - Covered: `shuttleengine/doc.go`, `claudeengine/doc.go` and `engine.go`.
- **`RoleOrch` is registered under the beam's slug, or `HubOwner` for the prime.**
  - Every `End`/`EndSparing` caller on those owners is affected.
  - Covered: the `fabriccli` anchor `EndDobbies` spare, with its test and doc.
  - Not wrong: tender reap (`tenderengine/processes.go:118-130`). `HubOwner` and an existing worktree's slug never read as orphan.
  - Not wrong: `hibernatecli` and `tendercli.EndOwnedElsewhere`. D9 licenses those, since the subject is owned elsewhere.
  - **Left wrong:** pair cancel (`pairteardown/cancel.go:166`), pair teardown (`teardown.go:335`) and `lym reed down` (`skip.go:94`, which spares only the dobby). On the beam's own slug, each now also ends the beam's orch Claude process. See F1.
  - **Left wrong:** the `beamcli/doc.go:423` Bound. See F1.
  - `runproc/doc.go:8-9` lists the `HubOwner` roles without `RoleOrch`. See F3.
- **New `EndSubject`, and the "only ways to end" list.**
  - Covered: `runproc/doc.go`, `end.go` header, and the PATTERN-child-processes Rule and bullets.
  - **Left wrong:** the PATTERN-child-processes Enforcement line. See F2.
  - The PATTERN.md index line has no list, so it is fine.
- **`OwnerWatch.End` field.** Its only constructions are `watch.go` and `watcher_owner_test.go`, both covered.
- **The way forward names `lym beam start`.** No shellguard row denies `lym beam start` to agents, so PATTERN-refusal-way-forward holds. The orch's subcommands include no `start` (`cli.go:184`).
- **Commit messages and departures.** No `Pattern-Departure:` trailer is in the range.
- **Declared PATTERN changes.** The directive declares none, so nothing is owed.
- **Stencil wording.** The diff changes no stencil.

**Findings**

- **F1.** MEDIUM, class `design`.
  - Location: `internal/orchcli/cli.go:296`. `RegisterOrch` under the beam's slug is unaccompanied by changes at `internal/pairteardown/cancel.go:166`, `internal/pairteardown/teardown.go:335`, `internal/pairteardown/skip.go:94` and `internal/beamcli/doc.go:423`.
  - Summary: the orch record shares the beam slug that pair teardown, cancel and `lym reed down` end through `runproc.End`/`EndSparing`. A cancel or teardown of the beam's landing run therefore now kills the beam's live orch Claude session even though the beam stays and is owned here.
  - D9 licenses ending the orch only for a subject owned elsewhere. The diff patched the same widening for `lym fabric anchor` (spares `RoleOrch`) but not for these callers.
  - Evidence: `cancel.go:166` `t.endOwned(slug)`; `skip.go:94` `t.endOwned(slug, runproc.RoleDobby)`; `teardown.go:697-698`, which spares only the given roles plus `RoleTeardown`.
  - Evidence: `beamcli/doc.go:423-426`, "runproc.End of the beam's own slug also ends its dobby. A cancel or an owner pause of the beam's landing run does this… `lym beam start` brings it back if the beam stays". It is now incomplete, since the orch is ended too.
  - The summary says pair-teardown ending "matches D9's 'end every process of a subject owned elsewhere'", which is wrong for a cancel or teardown on the owning machine.
- **F2.** LOW, class `consistency`.
  - Location: `pattern/PATTERN-child-processes.md:58`.
  - Summary: the Enforcement bullet still says "each end goes through `End`, `EndSparing` or `EndRole`". The Rule at lines 34 and 37, which the diff changed, now adds `EndSubject`.
  - Evidence: `pattern/PATTERN-child-processes.md:34,37,58`.
- **F3.** NIT, class `consistency`.
  - Location: `internal/runproc/doc.go:8-9`.
  - Summary: "The hub's own processes register under `HubOwner`. Their roles are `RoleSync`, `RoleSlackRelay`, `RoleTestMap` and `RoleAdopt`" leaves out `RoleOrch`. The prime's orch now registers under `HubOwner`; the diff's added sentence states this but does not amend the enumeration.
  - Evidence: `runproc/doc.go:8-9` and the added lines after them in the diff, plus `orchcli/cli.go:282-285`.

Tool calls:
1. Grep for the fallback, `abandoned_session` and end-function-list phrasing repo-wide, to find docs and fixtures outside the manifest.
2. Read `beamcli/cli.go:425-436` and grepped the beamcli doc, the `CheckResume` implementers, string matches on the old refusal text and shellguard's `beam start`, to complete the affected sets of the report field, error type and way forward.
3. Grep the `reedUp`/`startOrch` order and read `claudeengine/command.go:418-440`, to tell the `ResumeCmd` "falls back" doc apart from the orch launch.
4. Grep the pairteardown `endOwnedProcesses` callers and `RoleDobby` uses, to find ends on a beam's slug.
5. Read `pairteardown/skip.go:80-105` and grepped the `endOwned(` call sites, to confirm which roles cancel, teardown and `reed down` spare.
