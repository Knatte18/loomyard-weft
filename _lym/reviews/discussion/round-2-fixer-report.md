# Round 2 fixer report — discussion

The premise of every non-decision finding held against the code:
`CodeDirAt` lists direct files only, `internal/lymdirs` declares no function,
block keys span the whole `FuncDecl`, `emptyTierFailure` exempts only filter flags,
`prepare` reads `Status` once, and `computeScope` runs before `AcquireSlots`.

## Changes

- F1 (BLOCKING, fixed as proposed): D2 records, per linked package, every tracked file impactset maps to it:
  its own Go files and every other file under its directory outside a nested package directory, `go:embed` targets included.
  D5's last bullet compares that set.
  A D2 criterion and a D5 criterion cover an embedded file in a subdirectory of A.
  A Notes line says `CodeDirAt` lists direct files only.
- F2 (BLOCKING, fixed as proposed): D2 defines the linked packages as the whole coverpkg list, covered or not,
  and records and stales on all of them.
  D4 and D5 compare every linked package, and "covered package" is replaced by "linked package" throughout.
  The D2 staleness criterion now names a linked package with no covered block.
  The first Open risks bullet carries the cost over every linked package.
- F3 (BLOCKING, fixed as proposed): a Constraints bullet records the measured rate on main from 2026-11-04 to 2026-11-11.
  At least 95 of 118 commits that touch non-test Go still miss, and `internal/fabricengine` misses on 15 of 26.
  The source is a `go/parser` classifier of each changed file's top-level declarations against the first parent.
  D5 exempts an added const, a var with no initializer and a type with no methods.
  Changed or removed declarations, struct fields, var initializers and method-set changes stay misses.
  A D5 criterion says an added const in A is no miss.
  The rate still leaves `coverage` close to `diff`, so the third Open risks bullet is now a question for the parent tied to `testmap-upkeep` (support log Q11).
- F5 (MEDIUM, fixed differently): D4 compares the whole declaration, receiver, signature and body, ignoring comments and formatting.
  I did not add the proposed criterion "a value-to-pointer receiver change selects the B tests that covered it".
  F3's fix keeps a method-set change as a miss, and a receiver change from value to pointer changes the value type's method set.
  That can flip an interface assertion in unchanged code, so the package must run whole, not selected.
  A D4 criterion now covers a plain function's parameter-type change with the same body selecting.
  The receiver change is a D5 method-set miss, with its own D5 bullet and a D5 criterion.
- F6 (MEDIUM, fixed as proposed): a D5 bullet makes an added, removed or changed `//go:` line a miss and states the comment exception never covers it.
  The D5 criterion lists it.
- F7 (MEDIUM, fixed as proposed): D3's union keeps only the tier's tests, guards included, and drops watch-skipped tests.
  An empty union counts as `none selected` and is left out of D6's selected invocation.
  A D3 criterion covers the unwatched guard case.
- F8 (LOW, fixed as proposed): a D5 bullet makes a changed file that fails to parse at the map's `Commit` or in the working tree a miss.
  The D5 criterion lists it.
- F9 (LOW, fixed as proposed, bound option): D5's bound states that a converting checkout misses for every importing package.
  A support-log rejected alternative records why the clean-filter option was not taken (PATTERN-fabric-boundary).
- F10 (LOW, fixed as proposed): D2 repeats testmap's dirty check over every linked package's files after a package's tests end,
  and writes no map when one is dirty.
- F11 (LOW, fixed as proposed): the PATTERN-gate-slot constraint now says the scope reads maps before any tier takes its slots, and starts no build.
- Support log: Round 2 section, Q5 and Q6 updated, Q11 added, two rejected alternatives added.

## Self-check

Every fork returned a report.

- fix-proposal, BLOCKING: a const added inside an `iota` or implicit-value group shifts later values without a miss.
  Fixed: D5 compares such a group as one declaration, so an added spec is a change.
  The rationale and a D5 criterion carry it, and the measurement was rerun with that rule.
- fix-consistency, MEDIUM: D4's rationale still called every change outside function declarations a miss.
  Fixed: "a miss unless D5 exempts it".
- fix-consistency, LOW: the D3 guard and out-of-process criterion ignored the tier and the watch skip. Fixed in the criterion.
- fix-consistency, LOW: support log Q5 still said "covered package". Fixed.
- fix-claims, LOW: the count of 94 was stated as exact. Fixed: the classifier now sees `//go:` lines and `iota` groups.
  On the rerun, main had one more commit, so the count is "at least 95" of 118 against 103 before, with the in-module import approximation named.
- fix-claims, LOW: "discards that package's map" overstated what testmap does. Fixed: "writes no map for that package".
- fix-claims, LOW: the path-to-package restatement took in Go files under `testdata`. Fixed: the package's own Go files plus every other file.
- fix-way-forward, LOW: an ignored file the record lacks would miss forever. Fixed: the clause names files not ignored by git.

## Disputed

None.

## Deferred

None. F4 is a `decision` finding held by the parent and is left out of this report.

## Rulings

None reached me this round.
