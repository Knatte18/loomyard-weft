# Stop refusing marked tests of tiers a gate key does not list

A `lym gate test --gate <key>` call whose key has scope `marked` refused to run
when a test marked `//lym:only <key>` sat in a tier that none of the key's listed tiers is or covers.
The shipped `gate.gates.upkeep` is `{tiers: [0], scope: marked}`, while the tests marked `upkeep` here are tier1,
so a hub that kept that default, and whose tier0 does not cover tier1, got this refusal and ran no upkeep check:

```
gate test: --gate "upkeep" lists no tier that is or covers tier1, so its marked tests there cannot run: ...
```

The marked scope runs no package in its listed tiers;
its guard step runs each marked test under its own file's tier.
The coverage refusal is gone, so a marked test of a declared tier runs whatever tiers the key lists,
and no hub needs to set `gate.gates.upkeep` to reach tier1 tests.
The one exception is a tier declared `manual`: its marked tests run only when the key lists that tier
or one that covers it, and any other call is refused.

## Changes

- `internal/gatecli/only.go`: `markedScope` refuses a marked test of an unlisted tier only when that tier is `manual`.
- `internal/gatecli/scopes_integration_test.go`: the case that expected the refusal now expects the upkeep call,
  with the key still listing tier0 alone, to run both the tier0 and the tier1 marked test,
  the latter under tier1's settings.
  It also covers a `manual` tier1: refused for that key, run for a key that lists it or a tier covering it.
- `internal/gatecli/tiers.go`: the refusal of an undeclared tier names marked tests beside guard tests.
- `internal/gatecli/doc.go`, `internal/gateslot/template.yaml` and `pattern/PATTERN-test-speed.md` state the new rule.
  The template comment drops its advice to set `gate.gates.upkeep` to `{tiers: [0, 1], scope: marked}`.

## Design decisions

- The tiers are derived from the marked tests, and the template default stays `[0]`.
  A hub that declares tiers is refused when a key names one it does not declare,
  so a default of `[0, 1]` would break the upkeep call on a hub that declares only tier0.
- Among marked tests, a key's listed tiers now limit only those of `manual` tiers.
  A `manual` tier, such as one that spends tokens, runs only when named, so a test marker alone does not start it.
- A marked test of a tier the hub does not declare is still refused.
  So are a key with no test marked for it and a scanned directory without `go.mod`, in a hub that declares tiers.

## To check by hand

- `markedScope` (`internal/gatecli/only.go`) keeps refusing a marked test of an unlisted `manual` tier,
  and now makes that refusal for `manual` tiers alone; a listed tier that covers the `manual` tier allows the test.
  Check that this is the wanted rule.
- Not run: `lym gate test --gate upkeep` on the main branch of a running hub, with binaries that hold this change.
  Confirm that it runs the dead-code and .NET tests.

## Merge danger

**Door:** two-way
**Blast radius:** gate
