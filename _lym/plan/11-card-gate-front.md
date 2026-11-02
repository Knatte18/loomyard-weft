# Card 11 — card-gate-front

**Create:**
- `plan:internal/gatecli#strandRole` -> `func strandRole(strand string) string`
- `plan:internal/gatecli#callKeyDir` -> `func callKeyDir(call frontCall) string`
- `plan:internal/gatecli#TestCallKeyDir` -> `func TestCallKeyDir(t *testing.T)`

**Edit:**
- `internal/gatecli#detachRequest`
- `internal/gatecli#frontCall`
- `internal/gatecli#frontDecision`
- `internal/gatecli#front.writeNewRun`
- `internal/gatecli#runFront`

**Extends:**
- `internal/gatecli/doc.go`

**Uses:**
- `internal/gatecli#detachKeyDir`
- `internal/gatecli#cappedCall`

**Intent:**
Carry the calling strand's role from the front to the detached runner, and give a fixer's own gate call a detached run of its own (D4).

- The runner's environment carries no strand name, so `frontDecision` parses the role from the strand name it already reads, through `strandRole(strand)`, which returns `agentname.Parse(strand)`'s role and `""` for a name that does not parse, and keeps the strand and its role on `frontCall` (new fields `strand` and `role`).
- `detachRequest` gains `Role string` (`json:"role,omitempty"`); `front.writeNewRun` writes the call's role into it.
  A request with no role, as an older front writes, reads as a role that hands off, which card 12 still guards by the gate key's `fixer` and the pair slug.
- `callKeyDir(call)` returns `detachKeyDir(...)` over the call's verb, directory and words, as `runFront` computes it today, and for a call whose role `gatefixer.IsFixerRole` matches, adds the strand name to the hashed parts, so a fixer strand's own `lym gate test --gate card` call never joins the run waiting on it, and two fixer strands never share one.
  `runFront` keys its call with `callKeyDir`.

Bound: the role only selects whether a red verdict hands off (card 12) and keys a fixer's own call apart; it grants no call anything a non-fixer call lacks.

`TestCallKeyDir`, tier0: a non-fixer call's key equals `detachKeyDir` over the same parts; a `gate-fixer` call and a `gate-fixer-2` call with the same words get keys apart from it and from each other; a `verify-fixer` call is keyed apart; `strandRole` rows give the role of a well-formed strand name and `""` for a malformed one.

**ImpactSummary:** the front writes one more request field and keys only a fixer-role call differently; every other call keeps its key, its request shape and its join.
