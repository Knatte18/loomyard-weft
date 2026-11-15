---
round: 3
focus:
  - "Landing past the parent-side merge: in `_lym/discussion/decision-record.md` D1 (act rule), D4 (hand-over), D6 step 4, D8 step 3 and D9, settle whether one machine, and only one, finishes a Finalize past its parent-side merge after a takeover, and whether every one of those sections and the `PATTERN-owner-machine` constraint says the same."
  - "Pair push order: in D2 and D4, settle whether the old owner's first transition after a takeover can still move origin's code branch (a pair push sends the code side before the records side, and the takeover entry sits on records branches only), and what bounds the adopter's own code push."
  - "Network gate and pauses: in D2, D3 and D7, settle whether a pause or hold written between a transition's commit and its push leaves that transition unpushed at the pause record's network gate."
  - "Takeover race and local branches: in D4's last bullet and D8 steps 2 to 4, settle what an adopter re-reads before rewriting a takeover entry after a release and a second adopt, and whether the adopter's local branches reach origin's tips when the old owner pushed between the adopter's fetch and its takeover."
  - "Orch mirror: in D10, settle whether the last turns before a move reach origin, whether an old owner's late mirror can replace the new owner's newer local transcript at its next start, and whether cross-machine resume of a restored transcript is backed by recorded evidence with its source."
  - "Cancelled and landed batten records: in D1, D3 and D11, settle which shed state a finished cancel leaves, whether the dobby's done-run clear and `lym batten cleanup` can delete a kept cancel's task branch or refuse it forever, and what state a landed slug reads once its record went with its pair."
  - "Upgrade and teardown routes: in D6 step 4 and D7's upgrade bullet, settle which existing route removes a local pair without touching origin or the board claim, and whether a standing adopt pause entry written by past adopts stops every machine at the upgrade with a stated notice and way forward."
---

Foci cover the two BLOCKING design areas and the MEDIUM design areas of round 2.
The `decision`-class questions go to the run's parent and are not a focus here.
