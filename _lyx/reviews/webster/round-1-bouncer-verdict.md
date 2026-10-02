---
verdict: APPROVED
rationale: "Range main..HEAD over merge base 9ae5f9405 (Cards 1-2 plus seven fix commits); the round-1 report's five findings are all fixed in 2724f2bef, 7e44066c1, 655783fe4, 3f0c926d7 and the three F1 commits, the core evaluateGate/Wait counting matches Card 1, and go build, go test, go test -tags integration and go vet -tags smoke all pass at HEAD."
---

The review range comes from `_lyx/shed/gate-list/status.json`, whose `product.parent` is `main`.
The merge base is `9ae5f9405`, and the range holds the two card commits (`cdce74404`, `3f79defd0`) plus the seven fix commits the Burler round added on top.

The round-1 report carries no `## Focus departures` section, so there is nothing to ratify or reject.
Its five findings are each legitimate under the rubric, and each is now fixed in the diff:

- F1 (comment line breaks): a scan of every added or changed Go comment line in the range finds no mid-sentence break, bar one deliberate independent-clause break in `GateEntry.PassOnCap`.
  Two changed lines (`runlevel.go`'s "The seam is widened…" and `gate.go`'s `FindingsPath` "It is diagnostic text ONLY…") still start a sentence that continues on an untouched legacy line.
  Each sentence's text and break point predate the diff, and CONSTRAINTS.md's convention leaves untouched legacy wraps alone, so neither is a finding.
  The loom recipe's YAML comments are wrapped at a fixed column, but CONSTRAINTS.md scopes the convention to Go comments and the rubric forbids inferring a width from surrounding code, so they are out of scope too.
- F2: the `gate.go` file doc names all seven exported gate types and drops the count.
- F3: `GatesRejectedInsideProfileSubMap` plants `gates` inside `profile:` and asserts an unrecognized-key error naming it.
- F4: `resolveGateClosure` takes the element index, and its unknown-name error now reads `config key "gates" element <i>: …`.
  The new `UnknownNameNamesElement` case pins it.
- F5: the `loomrecipe` fake's `RunGated`/`AttachGated` doc describes the list evaluation through `evalGateList`.

I checked `evaluateGate` and Wait's gated-Done branch against Card 1 myself.
Entries run in order and stop at the first failure.
An off entry is reported off wherever it sits, and a capped `PassOnCap` entry is let through without running.
A not-reached non-`PassOnCap` entry makes `Passed` false.
Counts change only after a successful `Send`, and passed entries' failure counts reset then.
An exhausted non-`PassOnCap` entry finalizes on the memoised failed outcome.

Per-card mechanical check: Card 1's Create targets `shuttleengine#GateEntry` and `gate_list_test.go` exist, and so does Card 2's `entries_gate_test.go`.
Neither card has a `Rename` or `Delete` group.
I re-ran the plan's verify block at HEAD, and all four commands pass.
