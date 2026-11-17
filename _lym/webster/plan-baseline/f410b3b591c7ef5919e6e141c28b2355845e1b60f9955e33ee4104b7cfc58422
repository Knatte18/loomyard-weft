# Card 6 — owner-write-registers

**Edit:**
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#TestSetOwners`
- `internal/beamcli#hubCreateDeps`
- `internal/hubgeom#DobbyStart`
- `internal/adoptcli#hubDepsAs`
- `internal/battencli#hubOwnerSeams`
- `internal/orchcli#claimPrimeFor`

**Uses:**
- `internal/hubgeom/doc.go`
- `internal/adoptcli/doc.go`
- `internal/boardengine#EnsureIdentity`
- `internal/boardengine#Registry.Check`
- `internal/boardengine#RegisterMachine`
- `internal/boardengine#newOwnerFixture`

**Intent:** Register this machine at every owner write, in the owner write's own commit (D3: "at its first owner write"; "Only boardengine's registry setter writes the map, at a registration or an owner write").

`OwnerRequest` gains `Identity machinename.Identity`.
When `Identity.ID` is set, `writeOwners` runs `Registry.Check` against the loaded store's registry before touching any owner:
a refusal is returned and nothing is written, and otherwise the entry is applied as `RegisterMachine` applies it, in the same commit and push as the owner.
`req.Machine` must equal `Identity.Name` when both are set; a mismatch is refused as a wiring error naming both.
A request with no `Identity` registers nothing, as tests make it; `lym machine name`'s sweep carries the identity (card 10).
`SetOwners`' doc names the registration.

The production owner-write wrappers fill `Identity` from `boardengine.EnsureIdentity()`, and return its refusal (such as an unresolved config directory) as the seam's error:
`hubCreateDeps`' `SetOwners` seam (`lym beam create`), `DobbyStart`'s `ClaimPrime`, `hubDepsAs`' owner write (`lym adopt`), `hubOwnerSeams`' owner write (batten's prime claim) and `claimPrimeFor` (the bring-up's prime claim).
Each passes `Identity.Name` as `Machine`, so every owner write writes the current name.

`TestSetOwners`, tier1, gains rows: an owner write carrying an identity puts both the owner and the registry entry on origin when it returns; one whose identity a registered machine's current name refuses writes neither and returns the refusal.
The refusal row seeds origin's registry through `RegisterMachine` from the fixture's second clone (`otherDir`) under another id.

**ImpactSummary:** One new request field honored by the owner write; five production wrappers fill it, and every other caller of SetOwners is unchanged.
