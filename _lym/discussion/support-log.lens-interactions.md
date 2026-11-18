```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches.

**Findings**

1. **MEDIUM — D8 × D6: a view refresh during a push has no base to build from.**
   - D8 builds the view from "the merge base of branch and base", and the base is an argument the verb takes.
   - D8 then says "Fabric's push of a branch that has a local view ref refreshes the view in the same push". That push (D6) is handed only the branch, so it has no base.
   - D6's failed-push record "names the branch once", and nothing says what happens when the view's force-push fails while the branch push succeeds, or the other way round.

2. **MEDIUM — D7 × D6 and run-fsm: an owed prime push stalls prime runs until the operator pushes.**
   - D7 makes a prime records push "owed" whenever an operator commit is unpushed.
   - D6 and `PATTERN-run-fsm` say an owed push holds the run at its transition.
   - So any prime-anchored run, the batten Shed among them, stops until the operator pushes their own code. The record never says this and never bounds it.

3. **MEDIUM — D7 bound × D5 and landing: landing already pushes operator commits.**
   - D7's bound says "fabric never pushes ... an operator commit".
   - Finalize's local landing merges into the parent and then calls `PushBranch` on the parent, D5 unchanged. When the parent is the prime's branch, that push carries any unpushed operator commits.
   - The bound is false as written. The landing path either needs the same guard or the bound must exclude it.

4. **MEDIUM — D1 × D2 and D3: a broken one-repository hub reads as two repositories.**
   - D1's predicate needs `_bolt` present and sharing the prime's common git directory.
   - If `_bolt` is missing (a clone stopped before the Bolt worktree was added, or a damaged hub), the predicate reads "two repositories". `lymcwd.Resolve` (D3) then looks for a weft sibling that never existed.
   - D1's rationale covers robustness in the other direction only. The window inside clone, between cloning the prime and adding `_bolt`, is not covered either.

5. **LOW — D1 × Constraints (`PATTERN-cwd-resolution`): the predicate's home is unsettled.**
   - D1 puts the predicate in "a leaf package both fabricengine and lymcwd may import".
   - `PATTERN-cwd-resolution` limits lymcwd to the standard library plus `dotgit`, `lymdirs` and `weftname`. The predicate must live in one of those, or the entry changes.
   - `## PATTERN changes` lists no change, and D1 does not name the package.

6. **LOW — D7, push rule against pull rule: a landing merge is classified two ways.**
   - The push rule counts "a landing merge fabric made" as fabric-made.
   - The pull rule replays records-only commits and stops on "any other local commit", so it stops on that same landing merge.
   - Neither rule says how a landing merge is recognized: by trailer, author or message.

7. **LOW — D4 × D9 and the D4 criterion: "the prior code tip" is ambiguous.**
   - With records commits interleaved on the branch, HEAD before a transition commit can itself be a records commit.
   - The D4 criterion ("Warp-SHA naming the prior code tip") and D6's `OriginCodeTip` ("reads the branch tip") do not agree on which commit the trailer names: the immediate parent or the last commit that changed code.

8. **LOW — D8 × D6: the lock mode for the view push is not stated.**
   - D8 says "under the slug's teardown lock" without saying shared or exclusive.
   - `PATTERN-pair-remote` makes it shared for a pair push outside a removal. D6's teardown deletes the view ref, so the view force-push must take the lock shared to avoid racing the delete.

9. **LOW — D5 × D8, pull-request path: the local merge after a GitHub merge is not pinned.**
   - After GitHub merges the view, Finalize's local `Merge` of the task branch runs against a parent that holds the view commit. The view commit is not an ancestor of the task branch.
   - The three-way merge re-applies identical code changes. That is normally clean, but the outcome is not covered by an acceptance criterion.

**Pairs checked**
- D3 × D5 (each side's own anchor marker is stripped): held.
- D3 × D11 (shared `.git/info/exclude` with the Bolt): held.
- D3 × D4 (task inherits the parent's `_lym` at fork, as today's weft fork does): held.
- D4 × D9 (records commits move HEAD, identity read as code tree): held.
- D4 bound (mixed agent commits) × D5, D8, D9 (stripping by path): held.
- D4 × D10: held.
- D4 × D7 (records commits on the prime's branch, push guard): broken (findings 2 and 3).
- D7 push rule × D7 pull rule: broken (finding 6).
- D7 replay × the Warp-SHA trailer (the named tip stays an ancestor): held.
- D5 × D6 (hand-over, teardown): held.
- D5 × D11 (`lym-bolt` is never merged): held.
- D5 × D8 (merge-in moves the merge base, view recomputed): held.
- D5 × D8 (local merge after the GitHub merge): held weakly (finding 9).
- D6 × D8 (push refresh): broken (finding 1).
- D6 × D8 (teardown lock and view delete): unclear (finding 8).
- D9 × D5 (landed size uses the stripped diff): held.
- D1 × D2 and D3 (predicate during clone and on a broken hub): broken (finding 4).
- D1 × `PATTERN-cwd-resolution` import set: unsettled (finding 5).
- D2 × D11 (`--single-repo` seeding against weft-binding detection): held.

**Criteria and constraints checked**
- D4 criterion: ambiguous (finding 7).
- D5, D6, D8, D9, D10 and D11 criteria against their decisions: held.
- Constraint `PATTERN-gitrepo-client-boundary` (view force-push under the remote-refs lock): held.
- Constraint `PATTERN-github-auth`: held.
- Constraint `PATTERN-cwd-resolution`: finding 5.
- Constraint `PATTERN-pair-remote`: findings 2 and 8.

No further tool calls were made.
