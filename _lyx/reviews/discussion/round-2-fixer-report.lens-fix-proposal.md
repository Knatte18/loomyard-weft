```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging each reviewer proposal the fixer carried out.

**MEDIUM: F9's checkout way forward cannot wire at the target's anchor, and works only for one narrow kind of branch**

- **Changed text:** `decision-record.md` D10, the checkout way forward: "work on the target branch in a pair of its own, which wires at the target's anchor, such as `lyx fabric add &lt;slug&gt; --adopt-local` over a local branch of the slug".
- **Problem:** the review proposed a `lyx fabric add` pair. Under D5's own rule, that pair is wired at the fork point's anchor, not the target's. The command also only reaches unmarked local branches.
- **Evidence:**
  - D5 says "`Add` takes the new pair's anchor from the fork point, the parent's Location, or from `--anchor` (D6), never from a read of the not-yet-written weft".
  - `internal/fabricengine/doc.go` (AdoptLocal paragraph): "the weft side follows the usual rules, adopting a live records branch and otherwise forking from the parent's records branch".
  - So an adopted records branch whose marker differs from the fork point's anchor is wired at the fork point's anchor, over content that sits at its own marker. That is the half-rewired state D10's refusal exists to prevent.
  - The same paragraph limits `--adopt-local` to "an unmarked local warp branch that already exists" and refuses a branch checked out in another worktree. A target that is a marked pair branch, or one only on origin, has no route through the first option.
- **Change implied:** either give D5 a rule that an adopted live records branch takes its own committed marker as the anchor, or drop the "wires at the target's anchor" claim and the `--adopt-local` example from D10.

No other proposal broke under a concrete case:
- **F1/F3:** pass wait first, then parent lock; dobbies spawn only passes.
- **F2:** rename `.lyx` back first.
- **F4, F5:** refusals.
- **F6:** `U`-ancestor test, including when `A` is `B`.
- **F7, F8, F10–F13:** checked; none broke.

Tool calls:
1. Grepped `exec.Command`/`proc.Detach` in `internal/beamcli/dobbydeps.go` and `internal/dobbyengine`, and read the AdoptLocal paragraph of `internal/fabricengine/doc.go`. My context showed neither what children a dobby spawns (to test F3's in-flight read) nor how `--adopt-local` picks the weft side (to test F9's way forward).
