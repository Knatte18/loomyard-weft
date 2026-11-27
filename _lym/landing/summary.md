# Run a marked gate key's tests under their own tiers, not its listed ones

A `lym gate test --gate <key>` call whose key has scope `marked` refused to run
when a test marked `//lym:only <key>` sat in a tier that none of the key's listed tiers is or covers.
The shipped `gate.gates.upkeep` is `{tiers: [0], scope: marked}`, while the tests marked for it are tier1,
so every hub that had not set the key by hand got this refusal and ran no upkeep check:

```
gate test: --gate "upkeep" lists no tier that is or covers tier1, so its marked tests there cannot run: ...
```

The marked scope runs no package in its listed tiers;
its guard step runs each marked test under its own file's tier.
The listed tiers therefore served only as a check that the hub had copied the marked tests' tiers into the key.
That check is gone: a marked key now runs every test marked for it under that test's tier, whatever tiers it lists,
so no hub needs to set `gate.gates.upkeep` to reach tier1 tests.

## Changes

- `internal/gatecli/only.go`: `markedScope` no longer refuses a marked test of an unlisted tier;
  the helper that named the refused tests is removed.
- `internal/gatecli/scopes_integration_test.go`: the case that expected the refusal now expects the upkeep call
  to run the tier0 marked test under tier0 and the tier1 marked test under tier1, with the key still listing tier0 alone.
- `internal/gatecli/doc.go`, `internal/gatecli/cli.go` and `internal/gateslot/template.yaml` state the new rule
  and drop the advice to set `gate.gates.upkeep` to `{tiers: [0, 1], scope: marked}`.

## Design decisions

- The tiers are derived from the marked tests rather than defaulting the template to `[0, 1]`.
  A key that names a tier requires the hub to declare it,
  so a template default of `[0, 1]` would refuse every upkeep call on a hub that declares only tier0.
  Deriving them leaves the key's tiers unused by the marked scope while every hub runs its marked tests.
- A marked test of a tier the hub does not declare is still refused, through the guard step's existing refusal
  that names the undeclared tier.
- A key with no test marked for it, and a scanned directory without `go.mod`, are refused as before.
- Setting this hub's `gate.gates.upkeep` and the hand check of an upkeep run on the main branch are left out:
  they are configuration steps on a running hub, not repository changes.

## Merge danger

**Door:** two-way
**Blast radius:** gate
