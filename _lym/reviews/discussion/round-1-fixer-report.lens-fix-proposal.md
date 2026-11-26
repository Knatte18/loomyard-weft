```yaml
lens: fix-proposal
outcome: reported
```

Task: as the fix-proposal lens, find concrete cases where the review proposals the fixer carried out are wrong or break other parts of the record.

1. MEDIUM. Changed text: D13's `internal/fabriccli` paragraph ("is not bound … builds the seams … of both detached children … get their run-path, run-lock, status and takeover seams there"), together with the changed `PATTERN-fabric-boundary` line ("fabric learns nothing of IDEs, shuttle, shed, the board or heddle").
   Summary: keeping fabriccli out of the cut, as F1 proposed, moves the knowledge of shed's status and takeover codec into fabriccli, which is part of fabric. The fabric isolation rule forbids extending such knowledge.
   Evidence:
   - `pattern/PATTERN-fabric-boundary.md`, Fabric isolation: "Fabric is `internal/fabricengine` and `internal/fabriccli` … An import fabric already holds of such a module is debt. A change that touches it moves the knowledge out to the module, and never extends it."
   - The codec work in `codetipsettle.go` and `takeoverwrite.go` lives in fabricengine today. Building that seam in fabriccli for the teardown child and the push child extends fabric's shed knowledge rather than moving it out.
   - The new PATTERN line adds "shed" to the learns-nothing list, yet D13 makes fabriccli the place where shed's seams are built.

2. MEDIUM. Changed text: D13 shed bullet, "Rules that branch on a record … ask the seam's decoder for a verdict fabric defines", "Public readers (… `ReadUpstreamTakeover` …) return the record's bytes … and their callers decode them through shed", and "The result types that carried `shedengine.Takeover` carry fabric's own takeover entry".
   Summary: fabric decodes a takeover entry itself, not just a verdict, so "decoder returns a verdict" plus "readers return bytes" leaves no way to fill those result types.
   Evidence:
   - `internal/fabricengine/takeover.go`: `takeoverStop` calls `ReadUpstreamTakeover` inside fabric, and the `TakeoverError` it builds carries `Entry shedengine.Takeover` (the struct at `takeover.go:24-33`).
   - `handover.go:40` `HandOverRequest.Entry` carries the entry too.
   - The decoder must therefore return fabric's takeover entry (machine, subject, time) as data. Under the bytes-only reader rule the record leaves no stated path that fills those result types.

3. MEDIUM. Changed text: D13's required-seam bullet ("the run-records paths, the run locks, shed's codec and the stencil ordering judge … refuses before acting when it is unset") and its claim that fabric's tests may install "neither the real builder (D12) nor a stand-in for it".
   Summary: under F12's required-seam proposal, together with D12, fabric's own tests cannot run the takeover-write and code-tip-settle flows at all.
   Evidence:
   - The verbs refuse when the codec is unset.
   - D12 bars a fabric test from importing shedengine, so it cannot supply the real codec, and bars stand-in data for another module, so it cannot fake one.
   - D6 still requires "a tier1 seam test per flow shape" in fabricengine, and the new criterion pins those two flows' bytes.
   - The record does not say whether these flows' seam tests live in the shed module or how fabric's tests reach them. The same holds for any required path or lock seam, if a hard-coded shed path counts as stand-in data.

4. LOW. Changed text: D2, "Under a test binary `holdTeardown` runs the job in process on the parent's location; that path reads its own snapshot …".
   Summary: the in-process path also runs in production when `handOff` is false, so binding the per-child snapshot rule to the test binary alone leaves that production path unstated.
   Evidence: `internal/fabricengine/remoteteardown.go:340`: `if !handOff || testing.Testing() {`. The doc comment says "With handOff set, outside a test binary, a detached child takes the lock … Otherwise the lock is held in process".

Tool calls: one Bash call read `handover.go:28-48`, `takeover.go:24-36,112-140` and `remoteteardown.go:325-345`. My context showed only greps of those files, not the result-type fields or the full `holdTeardown` condition.
