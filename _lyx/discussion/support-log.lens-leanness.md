```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**MEDIUM — Notes for the plan writer: file pointers and call-site lists belong in `relevant.md`**
- The bullets list code locations: `finalize_owner.go`, `hibernateengine/wake.go`, `battencli/owner.go` and `adoptcli` "carry the … ways forward that change". The note "`beamcli` already imports `orchcli`" is also a code fact.
- The spec puts the list of code the writer read in `relevant.md`, never in the decision record. The rubric's Do-not-flag list also treats call-site enumeration as the planner's job.
- The completeness test passes: D4's last bullet and D6 already carry the requirement that every way forward naming a removed verb changes.
- Fix: cut these pointers, or move them to `relevant.md`. That file already lists `finalize_owner.go` and `wake.go`. Keep only the design hints:
  - pawl reuses beam's Finalize flags and needs a review-follows seam;
  - darn's perch is the model for pawl's perch;
  - the PATTERN background file needs rewriting.

**LOW — D6 Example: a walkthrough that repeats the rules above it**
- The G/H example replays rules D6's bullets already state ("a beam's own pawl run is resumed only by adopting its parent"), D10's land refusal and D5.
- Support-log Turn 6 holds the same walk, and Acceptance criteria already pin the behaviour.
- Fix: cut the example, or move it to the support log. No requirement is lost.

**LOW — Auto-mode assumptions repeat Decisions word for word**
- Every bullet restates a pick already stated in a Decision:
  - grouping projects follow a beam (D5);
  - `root` and `prime_owner` (D1, D5);
  - `lyx s` takes no slug (D4);
  - create runs from the parent worktree (D10);
  - the watcher stops the orch (D7);
  - Beam-Await waits in-process (D9);
  - legacy migration (D10).
- `lyx beam stop` taking no `--all` is the only one not stated elsewhere.
- The section is required, so don't cut it. Shrink each bullet to a one-line marker such as "D4: `lyx s` takes no slug (auto-pick)", and let the Decision hold the substance.

**LOW — Constraints: bullets that restate a PATTERN entry with nothing task-specific**
- Two bullets restate a PATTERN entry with no design content:
  - "`PATTERN-cli-cobra`: `lyx beam start`/`stop` are cobra verbs under the beam module";
  - "`PATTERN-stencil-ownership`: stencils read at call time".
- These are padding; the plan gate enforces PATTERN anyway.
- Fix: keep only the constraints that bind a decision here (config-layers moving to beam start, parent-lock, daemon-wakeups, tolerant-records, refusal-way-forward) and cut the rest.

**LOW — Open risks: the second bullet has nothing to act on**
- "A pawl perch reviews a whole project's diff … the cost is unmeasured" asks the parent nothing and changes no decision.
- Fix: cut it, or move it to the support log as deliberation.

**LOW — Goal repeats Scope**
- The Goal's second sentence lists the same items as Scope's In bullets: beam start, adopt, the pawl rows.
- Fix: cut the Goal to its first sentence plus "a beam's life is one batten run with the `pawl` recipe".

**LOW — D8 Rationale repeats the board entry**
- "a project is large, so its landing aligns and reviews…" copies the entry body.
- It is short, so this is minor; keep it or cut it.

The relocations above lose no requirement:
- The D4 and D6 refusal and way-forward rules stay.
- The D9 and D7 bounds stay.

Further tool calls: none; everything I needed was already in my context.
