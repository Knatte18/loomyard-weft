# Card gate vets the card's packages under the other tier tags

A card's gate (`--gate card`, tier0 only) never compiled the card's packages under the other tier tags.
So a helper defined in a `tier0` file and used by a `tier1` file broke only in the batch gate's `go vet` steps, far from the card that caused it.

Now, under the `go` scoper, a `--card` scope plans a vet step between the listed tiers and the guard step.
The step holds one vet run per declared tier whose tag `tierN` brings a `.go` file into a card package that a build without tags leaves out.
Tiers the call lists, and tiers a listed tier covers, are left out, since their own commands already compile under that tag.
A vet run is `go vet -tags tierN` over the card's packages, run once per module in that module's directory.

Closes #865

## Design decisions

- The step is part of the gate itself and needs no `gate.yaml` key.
  The `go` scoper already assumes `tierN` tag naming, so every hub's `card` key gets the step without a board edit.
- Only the `--card` form plans the step.
  The other gate keys in this hub list tier1, and tier1's command already vets each tag set.
  The `batch` key, also of scope card, now vets tier2 and tier3 tagged files a second time, a cost of mostly cached vet runs.
- A tier is vetted only when its tag changes a card package's file set.
  A package with no tagged file costs nothing.
  Bound: a break that shows only under a combination of tags is not caught at the card and is still left to the batch gate.
- A vet run is a normal run of the plan.
  It holds one gate slot and uses the tier's `env`, `parallel` and `timeout_min`.
  It runs with the go command the prebuild uses, goes through the `go-json` counter (which passes plain output through), and writes a run record and summary line with scope form `vet`.
  It carries through the green record under its own fingerprint.
  A failure stops no later run, and the closing line names it `vet tierN`.
- The tag scan (`tagTiers`) runs in `computeScope`'s card branch and is carried on the scope, so `planTierRuns` still never touches the disk.
- The `test` verb's Long help is at its 2000-byte cap, so the vet step is described only in `internal/gatecli/doc.go` (new `# Vet step` section) and `docs/overview.md`.

## Tests

- `TestPlanTierRuns` has new rows for the vet runs' order and modules, a tier a listed tier covers, and the `none` scoper.
- `TestGateTest_Scopes` (tier1) has two new rows.
  In one, the stand-in go records `vet -tags tier1` per module between tier0 and the guard step.
  In the other, real `go vet` fails a card whose tier1 file uses a tier0-only helper: the output holds `undefined: helper`, and the closing line names `vet tier1`.

## To check by hand

A live `--gate card --card N` in a hub task whose card touches tagged test files was not run.
