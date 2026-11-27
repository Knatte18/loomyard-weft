```yaml
lens: fix-affected-set
outcome: reported
```

Task: lens fix-affected-set, checking that the fix covered every member of each finding's set.

**Unchecked findings:** none. I searched for F1 (beam-cancel member claims) and F3 (owner-writer claims). I judged F2, F4–F9 from the context I already had.

1. **MEDIUM** — F3 fix, which amends only `internal/boardengine/doc.go` and the `owners.go` comment.
   - `pattern/PATTERN-owner-machine.md:29-30` still says: "An owner is recorded for a beam project as `owner_machine` … Only `boardengine`'s owner setter writes either."
   - `Board.EndBeam` writes `owner_machine` (`internal/boardengine/beamend.go:19`, `entry.OwnerMachine = ""`) and is not the owner setter.
   - The pattern now contradicts the amended package doc, which calls `EndBeam` "the one other write of an owner".

2. **MEDIUM** — F3 fix, a second twin left as it was.
   - The header of `internal/boardengine/ownerset.go:1` still calls the owner write "the only writer of a beam project's owner machine and of the hub's prime owner".
   - That is false for a beam project's owner machine, which `EndBeam` clears.
   - The sentence changed in `doc.go` ("the only writer that records an owner") shows the wording it needs.

F1: the search found no other beam-cancel claim left stale. The "move its `project` off the beam" text in `internal/landingshed/finalize_land.go` belongs to the landing refusal, which this fix does not change.

Tool calls:
- grep for beam-cancel member claims and `beam_ended` across Go, Markdown and YAML: my context held only the battencli sites.
- grep for owner-writer and sole-writer claims across the repo: my context held only `doc.go` and `owners.go`.
- read the `PATTERN.md` entry and `pattern/PATTERN-owner-machine.md` spans, and grep for `OwnerMachine` writes: my context lacked the pattern text and the other writers.
