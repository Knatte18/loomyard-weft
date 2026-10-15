---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/01-cli-and-output.md:29
    summary: The plan has the script's filters spell run-record and logs directory roots that card 1 says no filter spells, against PATTERN-lyxdirs-single-declarer as card 1 amends it
    fix: >-
      Make the three cards agree on how the filters get the directory names, with no new literal.
      In card 1, after the variable-block bullet that ends "...the records suffix", add the sentence "The block also derives the run-record root and the logs root as the first path segment of the plan directory and of the logs directory, with bash parameter expansion and no literal, and the script hands the plan directory and both roots to the jq filters with --arg; no filter spells any of them."
      In card 2, in the extract.jq paragraph, after "The `proj` field comes from the run name passed in, not from a hard-coded path prefix." add "The card count is taken from the prompt's card lines under the plan directory the script passes with --arg, not from a spelled `_lyx/plan/` pattern."
      In card 5, in the touched-files.jq bullet, replace "files under the run-record and logs directories and absolute paths outside the worktree are skipped" with "paths under the run-record root or the logs root the script passes with --arg, and absolute paths outside the worktree, are skipped".
      All three are prose edits; no target list or verify line changes, so no validate-plan overlay is needed.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-trace-cross-check.md:48
    summary: Card 4 makes runForkaudit append a different --records path on every call, which breaks card 2's two-run identical-report assertion once the header names that path
    fix: >-
      In card 4, add `plan:tools/forkaudit#TestForkauditReport` to `**Edit:**`, directly after the `plan:tools/forkaudit#runForkaudit` line.
      Then add to card 4's Tests section the sentence "`TestForkauditReport`'s run-twice row passes the same explicit `--records` path, one `filepath.Join(t.TempDir(), \"absent-records\")` computed once per row, on both runs, so the header's arguments line and records line match and the two reports stay identical once the `Generated:` line is removed."
      Keep the runForkaudit default for every caller that passes no --records.
      Checked with `lyx loom validate-plan --overlay` over card 4 with the extra Edit target, after the `{}` baseline: ok, no blocking finding, so the target change is admitted by every plan check.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/02-fork-records.md:49
    summary: Card 2 names only marker-table arms as empty arms, but its own --runs alpha row and card 3 expect unknown named and absent too
    fix: >-
      In card 2's `report.jq` Samples bullet, replace "an arm of the marker table with no forks is left out of the table and named once in a sentence under it" with "an arm among `file`, `raf`, `inline`, `none` and `unknown` with no forks is left out of the table and named once in a sentence under it; a `trace:<value>` arm exists only with forks".
      That matches the decision record, whose arm table lists unknown, and the `--runs alpha` row that names `inline`, `none` and `unknown`.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/01-cli-and-output.md:63
    summary: The read-only chmod rows never restore the mode, so t.TempDir removal fails on the marked read-only work directory
    fix: >-
      In card 1's Tests section, extend the line "Rows needing permission denial skip when `os.Geteuid() == 0`." with "Each such row registers a `t.Cleanup` that restores mode 0o755 on the directory it made read-only, before the chmod, so `t.TempDir`'s removal can delete the marker file inside the read-only `--work` directory."
      Card 2's chmod 000 row targets a file in a writable directory and needs no restore.
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/01-cli-and-output.md:27
    summary: Card 1 says every later card that adds a section updates the script header comment, but card 3 adds three report tables and does not edit the script
    fix: >-
      In card 1's header-comment bullet, replace "Every later card that adds a flag or a section updates this comment in the same card." with "Every later card that adds a flag or an optional section updates this comment in the same card."
      Card 3's default report tables are then outside the rule, as its target list already assumes.
---

### [BLOCKING] Filters spell directory roots the variable block does not hold

**Location:** `_lyx/plan/01-cli-and-output.md:29`, with `_lyx/plan/02-fork-records.md:45` and `_lyx/plan/05-pack-check.md:32`.

**Issue:** Card 1 amends `PATTERN-lyxdirs-single-declarer` to admit one variable block at the script's head, spelling `_lyx/webster/state.json`, `_lyx/plan/`, `.lyx/logs` and the records suffix, and states that no other line of `forkaudit.sh` or of a `.jq` file spells them.
The ports the other cards prescribe need more than those four strings.
`docs/research/glyph-audit-2026-10-15/extract-pf.jq:55` counts a fork's cards with the pattern ``- `_lyx/plan/\d+-``, and card 2 carries `cls`, `hit`, `segs`, `rtext` and the record fields over "unchanged" without saying where the plan directory comes from.
`docs/research/glyph-audit-2026-10-15/missed-files.sh:12` skips `_lyx/*|.lyx/*|/*`, and card 5 prescribes the same skip for `touched-files.jq` ("files under the run-record and logs directories") while the variable block holds neither bare root.
An implementer following the cards as written either spells `_lyx` and `.lyx` in a filter, which breaks the PATTERN line card 1 prescribes and the constraint that the literals sit once in the block, or invents a derivation the plan never states.
Round 1's F4 narrowed the single-spelling rule to the script and its filters, which is right, and it leaves exactly this gap open; no round-1 fix regressed and none was disputed.
The listed constraint wins over the port, so this is recorded as BLOCKING.
`plan-writer.jq` in card 6 is not affected: that card already says its pattern is "spelled from the variable block".

### [MEDIUM] Appended --records path differs per call and breaks the identical-report assertion

**Location:** `_lyx/plan/04-trace-cross-check.md:48`, with `_lyx/plan/02-fork-records.md:67`.

**Issue:** Card 4 changes `runForkaudit` to append `--records` "naming a path under a fresh `t.TempDir()`", and it makes the report header name the resolved `--records` path on every run; card 1's header also prints the exact arguments.
Card 2's `TestForkauditReport` row runs the script twice with the same arguments and asserts the two reports are identical once the `Generated:` line is removed.
Each `runForkaudit` call takes a fresh `t.TempDir()`, so after card 4 the arguments line and the records line differ between the two runs and that assertion fails under card 4's own `lyx gate test --tags integration ./tools/forkaudit`.
Card 4 does not list `TestForkauditReport` under `**Edit:**`, so nothing in the plan changes the row.
Card 2 cannot pass `--records` itself, since the flag does not exist until card 4.

### [LOW] Empty-arm naming rule excludes `unknown`

**Location:** `_lyx/plan/02-fork-records.md:49`, with `_lyx/plan/02-fork-records.md:68` and `_lyx/plan/03-arm-tables.md:29`.

**Issue:** Card 2 defines the marker table as `file`, `raf`, `inline`, `none`, with `unknown` as the fallback for a fork matching no row, and its Samples bullet names only "an arm of the marker table" that has no forks.
Its `--runs alpha` row then asserts the report names `inline`, `none` and `unknown` as arms with no forks, and card 3 asserts all three are absent from the tables.
By card 2's own rule `unknown` is never named, so the row cannot pass as written; the decision record's D3 table and D5 treat `unknown` as an arm like the others.

### [LOW] Read-only directory rows leave undeletable temp directories

**Location:** `_lyx/plan/01-cli-and-output.md:63`

**Issue:** The `--work` rows make a marked, existing `--work` read-only with `chmod 0555`, and the `--out` row makes the report directory read-only.
`t.TempDir`'s cleanup removes the tree after the test and cannot unlink the `.forkaudit-work` marker inside a directory without write permission, so the test fails in cleanup even though its assertions passed.
The plan names the root skip but no restore, and the testing rule that teardown leaves nothing behind needs one.

### [NIT] Header-comment update rule is broader than the cards follow

**Location:** `_lyx/plan/01-cli-and-output.md:27`

**Issue:** The sentence says every later card that adds a flag or a section updates the header comment.
Card 3 adds the pack-use, arm comparison and quartile sections and does not list `tools/forkaudit/forkaudit.sh` under `**Edit:**`, so the rule and the card disagree.
Cards 5 and 6 add flags and optional sections and do update it, so narrowing the rule to flags and optional sections matches all of them.
