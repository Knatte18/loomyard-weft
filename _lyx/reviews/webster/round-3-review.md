---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/yamlengine/doc.go:15
    summary: yamlengine comments still hold two clauses on one line (doc.go:15 and :20, fill.go:21, reconcile.go:44)
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/configengine/set.go:20
    summary: configengine set.go:20 holds two clauses on one line
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/configcli/configcli.go:152
    summary: configcli comments still hold two clauses or two sentences on one line (configcli.go:152, :190, :215, :224, doc.go:11)
  - id: F4
    severity: BLOCKING
    class: consistency
    location: internal/boardengine/config.go:30
    summary: boardengine comments still hold two clauses on one line (config.go:30, doc.go:15, :17, :20, store.go:767, :786)
  - id: F5
    severity: BLOCKING
    class: consistency
    location: internal/boardcli/cli.go:36
    summary: boardcli comments outside the previously fixed files hold two clauses on one line (cli.go:36, :269, doc.go:9, :11, :12, :22, :26)
  - id: F6
    severity: BLOCKING
    class: consistency
    location: internal/boardcli/help_test.go:148
    summary: new test comments hold two clauses on one line (help_test.go:148, store_test.go:1395)
---

Range: `_lyx/loom/rework/` has no round, so the range is `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` = `main` from `_lyx/shed/board-labels-and-skills/status.json`.
The round's focus is `PATTERN-comment-line-breaks`, so every finding below is that rule: a new or changed comment line with a semicolon, or a comma plus a coordinating conjunction (`and`, `but`, `or`, `so`), that joins two independent clauses on one line.
A line that ends at its semicolon, with the next clause on the following line, complies.
Compound predicates (`..., and refuses a shape mismatch`) and subordinate `which`/`where`/`that` clauses are not flagged.
Whether the plan's cards were implemented and their per-card mechanical checks ran is not judged here, because this round's directive narrows the review to the one rule.

### [BLOCKING] yamlengine comments

**Location:** `internal/yamlengine/doc.go:15`, `doc.go:20`, `fill.go:21`, `reconcile.go:44`
**Issue:** `doc.go:15` joins "so a legacy list survives" to the rest of the bullet with `, so`.
`doc.go:20` is `A list at P is an error naming P; the list is never converted.`
`fill.go:21` has `, so a list at that path against a mapping template is no mismatch;` before its semicolon.
`reconcile.go:44` has `, so both trees agree under it`.
**Fix:** Break each line at the clause boundary, one clause per line.

### [BLOCKING] configengine comments

**Location:** `internal/configengine/set.go:20`
**Issue:** `set.go:20` has `, so a key under one adds or rewrites an entry`.
**Fix:** Break the line before `so`.

### [BLOCKING] configcli comments

**Location:** `internal/configcli/configcli.go:152`, `:190`, `:215`, `:224`; `internal/configcli/doc.go:11`
**Issue:** `configcli.go:152` is `Look up the module; its declared open maps ...` and `:215` is `Handle --print ...; the print path is read-only`.
`:224` has `, so a module is always present here`.
`:190` is a changed line that holds two sentences: `without opening an editor. The print path is evaluated before any edit logic.`
`doc.go:11` is `It refuses when the map holds a list, and removing an entry stays an editor edit.`
**Fix:** One clause per line at each site.

### [BLOCKING] boardengine comments

**Location:** `internal/boardengine/config.go:30`; `internal/boardengine/doc.go:15`, `:17`, `:20`; `internal/boardengine/store.go:767`, `:786`
**Issue:** `config.go:30` has `, so the tags are inert`.
`doc.go:15` joins `a note carries exactly one of them` and `a task carries one or more` with `, and`.
`doc.go:17` and `doc.go:20` each have `, so <subject> <verb>`.
`store.go:767` joins two independent clauses with `, and`, and `store.go:786` has `, so a merge never loses a recorded issue`.
**Fix:** Break each line at the clause boundary.

### [BLOCKING] boardcli comments

**Location:** `internal/boardcli/cli.go:36`, `:269`; `internal/boardcli/doc.go:9`, `:11`, `:12`, `:22`, `:26`
**Issue:** `cli.go:36` is `config is the Config PersistentPreRunE loaded; the labels verb reads ...` and `cli.go:269` has `, so help and validation cannot drift`.
In `doc.go`, `:9` has `, so the config is path-only`, `:11` joins two independent clauses with `, and`, `:12` and `:22` use a mid-line semicolon, and `:22` also has `, and an absent target is an error`.
`:26` is a three-clause comma splice: `Each list is in ... order, an empty list is `[]`, and a list-shaped ...`.
These files were not among the sites the earlier rounds fixed.
**Fix:** One clause per line at each site.

### [BLOCKING] test comments

**Location:** `internal/boardcli/help_test.go:148`, `internal/boardengine/store_test.go:1395`
**Issue:** `help_test.go:148` has `, so the sentence cannot drift from validation` and `store_test.go:1395` has `, so the existing entry's own list leads`.
**Fix:** Break each line before `so`.

## Focus departures

None of the findings departs from the directive.
`internal/boardcli/cli.go` lines 615–652 carry a comment wrapped at a fixed column, which the diff only re-indents (`git diff -w` shows no change), so it is untouched legacy and is not a finding.
