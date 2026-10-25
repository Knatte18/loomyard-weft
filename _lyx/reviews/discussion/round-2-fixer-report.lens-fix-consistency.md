```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the fix's changes still agree with the sibling decisions, criteria and cross-references they touch.

1. **MEDIUM** — D10's new checkout way forward contradicts D5.
   - Changed text: D10's checkout way forward, "work on the target branch in a pair of its own, which wires at the target's anchor, such as `lyx fabric add &lt;slug&gt; --adopt-local` over a local branch of the slug", and the matching acceptance criterion, "the checkout naming a pair of the target branch's own".
   - The problem: D5, which the fix left as it was, says "`Add` takes the new pair's anchor from the fork point, the parent's Location, or from `--anchor` (D6), never from a read of the not-yet-written weft".
     With `--adopt-local`, `Add` "adopt[s] a live records branch" (`internal/fabricengine/doc.go`, the AdoptLocal paragraph).
     That adopted weft already carries the target's own marker, so under D5 `Add` would wire the pair at the parent's anchor, over content that sits at the target's anchor.
   - The result: the way forward does not wire "at the target's anchor" as D10 now claims.
     D5 needs a twin rule: an adopted, existing records branch takes its anchor from its own committed marker, or D5's markerless value.
     Otherwise the way forward has to name something else.

2. **LOW** — D7's prime refusal still gives a reason that the next line denies.
   - Changed text: the D7 prime refusal bullet, rewritten to drop creel and add the sentence "The creel daemon is not refused: its state sits under `&lt;hub&gt;/_board/.lyx/creel`, outside every anchor".
   - The problem: the bullet's lead-in still reads "since the hub's batten records, gate slots, orch and daemon state sit under the prime's anchor".
     That keeps the generic "daemon state" premise which the new sentence contradicts for creel.
     The lead-in should name the hibernate daemon's state only.

Every other twin I checked agrees with the change set:
- the D7 acceptance criteria (refusals, the parent lock hold, what is spared, `.lyx` kept on recovery)
- step 8's ordering against "A kill after step 8 removed the record"
- D8's refusal of a pair with a standing record
- D10's refusal while a record stands, against D7 step 1
- D2's empty-marker list
- the support log's round 2 summary

Tool calls:
- One Bash call read `internal/fabricengine/doc.go` lines 665–690 (the AdoptLocal paragraph), which my context lacked, to check what `--adopt-local` does to the weft side.
- The same call grepped the decision record for "daemon state" and "hibernate", to find the line numbers of the refusal's lead-in.
