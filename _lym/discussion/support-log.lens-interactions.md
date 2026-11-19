```yaml
lens: interactions
outcome: reported
```

Task: review how the decision record's decisions interact, as the `interactions` lens.

**Findings**

1. MEDIUM — D2 × D3 (layout cache during clone). D2 caches the layout answer for each records store for the life of the process. It also reads a path with no store on disk as two repositories. A `--single-repo` clone runs in a single process: `CloneHub` and then `CloneAndWire`.
   - If anything reads the layout of the weft-sibling path before D3 has created the records store and set its refspec, the cached answer is "two repositories".
   - Every later explicit push in that same clone would then skip the translation, so the board and records branch go to origin's `refs/heads/`.
   - That would break D1's acceptance criterion "leaves every `refs/heads/` ref at its prior tip".
   - The record never says a no-store answer stays uncached, and never says clone avoids reading the layout before the store exists.
   - Evidence: D2 says "reads it once per records store per process and caches the answer" and "a path with no store on disk … reads as two repositories". D3 creates the store partway through the clone.

2. MEDIUM — D3 refusal × D3 detection (restart after a partial namespace push). D3 pushes two things into the namespace: the board and the prime's records branch. Suppose the board ref lands and the records-branch push is refused.
   - D3 then removes the hub directory, but origin now holds the board branch under the namespace.
   - A rerun of `--single-repo` hits D3's own refusal "origin already holding the namespace". Its way forward is "the same clone without `--single-repo`".
   - That clone takes the namespace path (D3 detects the layout from the board branch) over an origin that lacks the prime's records branch.
   - So the record's own "nothing half made" rationale does not hold on this path.
   - In two-repository clones, `CloneAndWire` already pushes the Bolt before the per-worktree commit (the `b.PushRecorded` step). D1 makes the one-repository flow identical, so this ordering is the default.
   - Fix: push both refs with one atomic push, or push the records branch before the board, which is the detection key. Either way, state the order.

3. MEDIUM — D10 × D3/D1 (re-cloning an f06acf39 origin). D10 says an f06acf39-layout hub "is cloned again with D3". That origin's code branch still tracks `.lym-anchor` at the root and records under `_lym`, and origin still holds `lym-bolt`.
   - `--single-repo` passes every D3 refusal over it: there is no namespace, and the default branch has no `.lym-warp`.
   - The prime then checks out a code tree with a tracked `_lym` directory, where the D1 wiring wants to put a junction. Records stay on the code branch.
   - That contradicts D1's goal and the D2/D5 criterion that a merged code branch has no `_lym` path.
   - D10 needs to say either that the operator removes those paths and `lym-bolt` first, or that D3 refuses an origin whose default branch tracks `.lym-anchor` at the root. A cheap refusal of that kind is the sort `drop-compat-migration-code` keeps.

4. LOW — D7 acceptance × D1/D2 (the probe criterion targets the wrong store). The criterion reads: "a status commit during a cherry-pick in the code worktree commits instead of skipping".
   - Records commits go to the records store, so a cherry-pick in the code worktree never touched them. The criterion would pass whether the probe is reverted or not.
   - The regression T4 describes is a cherry-pick in the weft worktree during fabric's own replay.
   - Point the criterion at the records worktree.

5. LOW — D6 × D7 and acceptance (protected refusal: class and retry). D7 says it reverts f06acf39's two-repository widenings. D6 keeps one of them: a protected refusal is named and not retried, on every branch.
   - The D5/D6 acceptance criterion says the refusal "records the ordinary rejection class, and the retry does not retry it".
   - Before f06acf39, an ordinary rejection was retried, so this is a kept widening. D7's list does not say so.
   - Name it in D7 as kept-by-D6, so a reviewer reading D7 does not revert it.

6. LOW — D1 refspec × D4/D2 (bare push sends every tag). D1 configures the push refspec `refs/tags/*:refs/lym/tags/*`. Under it, a bare `git push origin` pushes every local tag; the support log, turn 2, records this.
   - D4 and D2 rely only on explicit, translated ref names. Nothing in the record forbids a bare push from fabric or states the hazard.
   - D9 lists the mirror hazard but not this one.

**Pairs checked**
- D1 × D2 (refspec as the layout): held.
- D1 × D3 (`.lym-warp` names the one URL vs the default-branch `.lym-warp` refusal): held. The board's `.lym-warp` lives under the namespace, not on the default branch.
- D1 × D4 (namespaced branches and tags, fetch refspec returning tags): held.
- D1 × D9 (hazards): partly; the bare-push tag hazard is missing (finding 6).
- D2 × D3 (cache vs a store created mid-process): broken (finding 1).
- D2 × D4 (translation applied to explicit fetch, push, ls-remote and delete; teardown by full name): held.
- D2 × D5 (no layout input outside fabric; seams removed): held.
- D2 × constraint `PATTERN-cwd-resolution` (anchor at the weft root): held.
- D3 detection × D3 refusal and restart: broken (finding 2).
- D3 × D8 (builder drives the D3 clone): held.
- D3 × D10: broken (finding 3).
- D5 × D6 (removal vs the general keep rule): held.
- D5 × D7 (commit refusal order, merge probe): held.
- D6 × D7 (protected-refusal widening kept vs "revert widenings"): unstated (finding 5).
- D7 × the peer `two-repo-merge-probe-regression`: held; Open risks lists it.
- D7 acceptance × D1: weak (finding 4).
- D1 acceptance (`refs/heads/` untouched) × D3 "no commit to any code branch": held, given findings 1 and 2.
- D2/D4 acceptance (records-branch and archive-tag placement) × D4 teardown: held.

No further tool calls made.
