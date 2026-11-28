# Admit a beam's orch as its members' parent and name it in their prompts

A beam's orch owns its members' runs and its own beam's project record, but several caller checks refused it.
Each check admitted the prime's orch, or one name a liveness probe had picked.
This change fixes three such gaps.

## What changes

- **Decision and parent-review verbs.**
  `lym loom decision add`, `replace` and `store` and `lym loom parent-review answer` now admit every orch on the run's beam chain.
  The chain is read from the origin records, and no orch is asked whether it is live.
  For a task run that is the orchs above the run's pair.
  For a beam's project record it is the beam's own orch and the orchs above it.
  Before, each verb admitted one name: the nearest orch up the chain that the liveness probe reported live.
  When the probe passed over the beam's orch, the beam's orch was refused as "a task session"
  or as "neither the run's parent orch nor the strand the ask went to".
  A refusal still names the nearest live orch as the one to ask.
- **Tender caller rules.**
  `lym tender pause`, `resume` and `revive` read a beam's orch as a caller kind of its own.
  It may act on its beam and on every pair whose beam chain holds the beam.
  It is still refused a hub-wide call, a pair outside its beam, and `reap`.
  Before, only an orch name with no slug counted as the orch, so a beam's orch fell under the agent rule, which admits only the agent's own pair.
- **Tackler and arbiter prompts.**
  Both prompts now render a new hand-back variant of the parent directive, `parent-directive-handback`.
  It names the orch the run hands its stops back to.
  Before, both rendered the no-parent text "No parent is recorded for this run" for every run.
  The verdict file stays their only channel, and a run with no parent keeps the no-parent text.
  A loom run names its pair's recorded parent.
  A batten run is driven from the prime, so it names the orch of the parent `hubgeom.ParentOf` answers for its slug.

## New helpers

- `hubgeom.OrchChain` lists the orchs on the beam chain at and above a worktree, nearest first and the prime's last.
  It walks the same chain `NearestLiveOrch` walks, ending at a gone parent, a missing record or a cycle.
- `hubgeom.ParentOrchName` names the orch of an entry's parent for a run driven from the prime.
- `parentdirective.HandBack` renders the hand-back or no-parent variant.

## Design decisions

- The caller checks drop the liveness probe and admit the whole chain.
  They do not make the probe more accurate.
  An orch on the chain owns the run whether or not a probe sees it.
  The chain always holds the nearest live orch, so every caller admitted before is still admitted.
- Notice and ask routing (`NotifyParent`, `creel post --parent`, the dobby's notices) still goes to the nearest live orch.
  That routing is meant to keep mail off an orch that is down.
  The fault report gives no reproduction of why the beam's orch read as down, so this change leaves the routing alone.
  The checks above no longer depend on it.
- The tackler and arbiter do not get the full parent directive, which tells a role to message its parent and wait.
  Both are spawned verdict-only, and a batten tackler's pane runs in the prime, where `creel post --parent` would reach the prime's orch.
  The hand-back variant names the parent without opening a second channel.
- The driver prompt already names the pair's recorded parent, so it is unchanged.

## Tests

- `TestPauseAndResume_ApplyTheCallerRules` adds a beam and a member forked from it.
  The beam's orch pauses, revives and resumes both, and is refused a hub-wide pause or revive, another pair and `reap`.
- `TestDecisionVerb_AppendsAndCommits`, the decision store table and `TestParentReviewAnswerVerb` add rows.
  In each, a beam's orch on the chain is admitted while the prime's orch resolves as the parent.
- `TestChainWorktrees` covers the walk behind `OrchChain`: member to beam to prime, a gone parent, a cycle, the prime, and a read error.
- `TestParentOf` checks `ParentOrchName` for every parent shape it covers.
- `TestHandBack`, `TestTacklerPrompt` and the arbiter's `TestPrompt` cover the hand-back text with a parent and the no-parent text without one.

## Merge danger

**Door:** two-way
**Blast radius:** orch
