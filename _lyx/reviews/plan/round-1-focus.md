---
round: 1
focus:
  - "Granularity of `17-merge-in-row.md`: its Edit list spans loomshed, shedrecipe, loomrecipe, landingshed, loomcli, contracts/stencils and refusal-spec. Settle whether it is one independently reviewable unit or bundles separable units, and whether the test-glyph edits belong with the code they change."
  - "Fidelity to D14 in `decision-record.md`: `PATTERN-parent-lock` appears only in `19-guarded-landing-rows.md`. Settle whether some card also carries the `PATTERN-told-geometry` bound-package listing and the `PATTERN-wait-mark-display-only` caller-set change, and whether card 19 is the right place for the PATTERN entry given the cards 1-18 that already introduce the lock."
  - "Fidelity to the Constraints and Decisions D1-D13 in `decision-record.md`: for each, settle which card carries it, in particular D7 (idle bound), D10 (batten resume on a moved base), D11 (lock verbs), D12 (status wait) and the `hubreconcile.Ensure` and `PATTERN-hub-wide-config` constraint for `parent_lock_idle_release_min`. Also settle whether any card adds scope the decision record does not license; the carry-over block licenses none."
  - "Card order against the Plan vs. schedule rule: cards 12, 14 and 16 use symbols that cards 15, 17 and 18 introduce or wire later (merge config keys, `NameMerge*` constants, the Merge-In row). Settle whether each card's `Uses` depends on a later card."
  - "Reachability of prescribed refusals and Stuck exits (dirty, mid-merge or missing parent pair in card 7 or 11; unreadable lock record and paused lock wait in cards 2 and 20; the three-reroute cap in cards 3 and 4; batten lock verbs in card 24). For each, settle whether a named input and a named test reach it, and whether an upstream check rejects that input first."
  - "Attack-surface question: `skip_merge_review` (card 18), `lyx batten lock release` and force release (cards 1 and 24), the idle-release bound and the batten backstop (card 23), and the parent-pair fast-forward (card 7). Settle whether each card's `**Intent:**` states what the change can skip or let through and what bounds it."
  - "Member glyphs and the re-sign arrow on `Edit` cards that change existing signatures, such as `Publish.Call`, `NewPublish`, `Deps` fields and the loop hooks in cards 5, 11, 17, 19 and 20. Settle whether each lists member glyphs with the arrow where the `Intent` changes parameters, results or receiver."
  - "Test economy across the large Edit-test lists in cards 17, 19 and 22: settle whether any new test function duplicates behavior an existing test named in the repo already covers, naming that test."
  - "Verify coverage: the `## verify:` section runs `go test ./...` and `go test -tags integration ./...`. Settle whether every package the cards target, including `contracts/stencils`, `internal/loomcli` and the board-side batten packages, is run by these commands, and whether the section's order matches the plan template."
---

Seed focus only; every entry names where to look and leaves the verdict open.
Mechanical checks, missing `DependsOn`/`Produces`, `ImpactSummary` on non-Edit/Delete cards, and `support-log.md` are excluded per the rubric.
