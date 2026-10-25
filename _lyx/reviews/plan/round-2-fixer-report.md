# Fixer report — plan, round 2

## Changes

- **F1 (MEDIUM, design)** — fixed as proposed, with one wording change.
  I swept the non-Go files outside `docs/research` and `docs/benchmarks` for the names the plan removes: `parent_review_wait_min`, `skip_alignment`, `alignment_timeout_min`, `parent-review`/`parent review`, `parentreview`, `lyx loom review`, `pr-gate-loom`, `template-alignment`, `alignment agent`, `ReviewWait`, `merge_fix_attempts` and a case-insensitive `alignment`.
  Every hit is a target of a card except the two lines the review named.
  - `_lyx/plan/06-retire-alignment-keys.md`: a `**Prosa:**` group holding `docs/overview.md`, placed before `**Delete:**`.
    A new Intent bullet says the loom config module paragraph drops `parent_review_wait_min`, and that the reconcile paragraph after it names the four retired keys among the keys it drops.
  - `_lyx/plan/04-drop-review-verbs.md`: the Docs bullet now also says the `status` waiting-note sentence names a verify and a shuttle wait, and no longer Discussion-Write's parent-review gate.
  - Different from the proposal: the bullet names the paragraphs by their text, not by line 569 and 570.
    Card 5 removes two module-tree lines earlier in the same file, so the line numbers would be wrong by the time card 6 runs.
- **F2 (LOW, design)** — fixed as proposed, and extended by the sweep the fix asks for.
  I searched the test files for the literals `Merge-In` and `merge-in` and mapped each hit to its enclosing declaration.
  The sweep found four more hits that must change, beyond the two the review named:
  - websterengine's `verifyGateConfig`, whose `gates` block keys `merge-in`.
  - `newMergeInFixture`'s gate key.
  - `TestMergeIn_PlanVerifyFailureRecord`'s `fixedGateTierLists["merge-in"]` lookup and its wanted `Kind`, which is `verifytree.FailureKind(g.key)`.
  - `TestMergeIn_VerifyGate`'s wanted site label, which is the producer name.

  The changes to `_lyx/plan/16-parent-align-row.md`:
  - **Edit:** gains `internal/gateslot#TestLoadConfig`, `internal/loomshed#roundGateConfig` and `internal/websterengine#verifyGateConfig`.
    All three resolve with `lyx quarry resolve`.
  - The three `mergein_test.go` members are named in the Intent only.
    Listing them under **Edit:** made `lyx loom validate-plan` refuse with `redundant-file-target`, because the card's Rename pair already covers that file, as it covers `TestMergeIn_Fixer`.
  - The fixture sentence names every new hit.
    It now says the list comes from that search, and that fabric-verb, git-merge and free-form row-name literals keep their text.
  - The earlier `verifyGateConfig` mention now reads "webstercli's", so the two functions of that name are told apart.
- **F3 (LOW, design)** — fixed as proposed.
  `_lyx/plan/12-bouncer-decision-escalation.md` gains, before **ImpactSummary:**, a paragraph opening "What it can skip or let through:" and one opening "What bounds it:".
  The self-check widened the retired-decision bound; see the self-check section below.
- **F4 (LOW, design)** — fixed as proposed.
  `_lyx/plan/07-scope-lenses.md`'s first Tests bullet keeps only the six-entry order assertion for `review-discussion`.
  The added sentence checks for `goal-scope` and `board-fit` are gone, and the `TestLoad` case stays.

`lyx loom validate-plan` over the changed plan gives the baseline result: `ok: true`, with only the informational `create-new-unit` finding on card 12.

## Self-check

- fix-consistency, LOW: card 6's ImpactSummary did not name the new `docs/overview.md` edit.
  Fixed as proposed: the ImpactSummary now names the overview's loom config and reconcile paragraphs.
- fix-claims, MEDIUM, and fix-proposal, LOW (merged; both report the same gap): card 12's "What bounds it" counted only an accept retired at the budget below the hard cap.
  The card's own pending-`continue` bullet and test row 12 also re-escalate a `continue` that meets the hard cap at settle time.
  Fixed as proposed: the bound now reads "plus one per decision retired, either an accept held at the budget below the hard cap or a continue that meets the hard cap at settle time".
- fix-affected-set: no findings.
  Its reruns of the F1 and F2 searches found every hit covered or correctly left alone.

## Disputed

None.

## Deferred

None.
