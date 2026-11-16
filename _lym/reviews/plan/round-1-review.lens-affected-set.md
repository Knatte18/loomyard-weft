```yaml
lens: affected-set
outcome: reported
```

Task: the affected-set lens. For each change a card makes, find what reads or asserts it, and which cards cover that set.

**Affected sets per change**

1. **Card 1, new `machinename.HostnameName`.**
   - Set: `internal/machinename/doc.go`, `TestSoleSource_NoOtherProductionFileReadsTheHostname` and `TestFromHostname`.
   - The package doc already says the package is the hostname's only production reader, and that stays true. The scan's exclusion of `internal/machinename/` covers the new reader.
   - Covered by card 1. Nothing outside the card is forced.
2. **Card 2, new `OwnerRequest.Replace` field.**
   - Set: every production `OwnerRequest{` literal, at `beamcli/create.go:163,181`, `adoptcli/steps.go:198`, `hubgeom/dobbygeom.go:109`, `orchcli/ownerpause.go:138` and `battencli/owner.go:197,215`. All are keyed literals, so the new field leaves them unchanged.
   - No test carries a `//lym:seam` marker for `OwnerRequest`.
   - Covered by card 2, which needs no caller edits.
3. **Card 2, new commit subject `board: owner <machine> replacing <names>`.**
   - Set:
     - `ownerset.go:105`, which builds the subject.
     - The `wantSubject` rows of `ownerset_integration_test.go`: existing rows keep the old form, and the new rows are on card 2.
     - `internal/boardengine/doc.go:174`, which states the commit as `"board: owner <machine> for <slugs>"`.
   - Card 2 covers `ownerset.go` and the tests. It does not name `doc.go:174`.
4. **Card 2, replace selection (owners read as another machine's get rewritten).**
   - Set: `doc.go:175` ("The commit sets each named beam project's owner, the prime owner when asked") and `doc.go:183` ("Without Takeover … leaves … untouched").
   - Also `doc.go:185` ("Only `lym adopt` passes Takeover"), which stays true.
   - Also `adoptcli/doc.go:273`, which is unaffected.
   - Also `pattern/PATTERN-owner-machine.md:27` and the `PATTERN.md:24` line.
   - Card 2 names `:183`, the pattern bullet and the `PATTERN.md` line, which is fenced as required by `pattern-line-unprescribed`. It does not name `:175`; see finding 1.
5. **Card 3, new module `machine`.**
   - Set: the root `Long` "Available modules" list (`cmd/lym/main.go:135`), the `AddCommand` list, `cmd/lym/registration_test.go` and `cmd/lym/clitree_test.go`.
   - Also the deviation list in `PATTERN-cli-cobra`.
   - `cliwire`'s enforcement only polices `webstercli` and `burlercli` (`bannedecl_enforcement_test.go:22`).
   - No skill, spec or settings file names `lym machine`.
   - The existing ways forward that already point at `lym machine name <name>` are now satisfied:
     - `machinename.go:28`
     - `beamcli/dobby_test.go:163`
     - `battencli/predrive_test.go:186`
     - `battencli/owner_test.go:107`
   - Covered by card 3.
6. **Card 4, sweep and recovery output.**
   - Set: `internal/machinecli/doc.go` only. The new package has no other readers.
   - Covered by card 4.

**Findings**

1. **Severity: LOW. Class: consistency. Location: card 2 (`02-owner-replace-set.md`), its `internal/boardengine/doc.go` edit.**
   - Summary: card 2 changes the owner commit's subject and adds replace-set selection. Its doc.go instruction names only the `:183` sentence and "the section states the selection rule". It misses two sentences its change makes false:
     - `doc.go:174`, which pins the commit as `"board: owner <machine> for <slugs>"`. Card 2 adds the `replacing <names>` form.
     - `doc.go:175`, which says the commit sets each *named* beam project's owner and the prime when asked. A replace-selected entry is neither named nor asked for.
   - Evidence: `internal/boardengine/doc.go:174-175` and `:183`; `ownerset.go:105`; card 2 lines 30 and 42.
   - Fix: card 2's doc.go paragraph names `:174` and `:175` as well, so the stated commit form gains the `replacing` variant, and the sentence about what the commit sets gains the replace-selected entries.

**Tool calls**

1. Grep for `machinename/doc.go`, commit-subject readers and the adopt-only clause: the manifest held no `machinename/doc.go`, and nothing searched for readers of the subject or the clause.
2. Grep for `//lym:seam` markers, module-list copies and the registration oracle: the manifest held no reflective seam markers or other module-list sites.
3. Grep for caller-set guards and `cliwire` enforcement scope: needed to see whether the new `machinecli` imports trip an enforced caller set.
4. Read `cmd/lym/gitrepoboundary_test.go`: needed to confirm that its `Pull` and `HasUnpushed` entries pin `gitrepo` methods, not callers of `fabricengine`.
5. Grep for golden copies of "Available modules" and the production `OwnerRequest{` literals: the manifest held neither.
