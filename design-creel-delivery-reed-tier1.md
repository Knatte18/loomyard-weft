# A tier1 creel delivery backed by real reed, so tests stop listing strands by hand in creelfake.Delivery

`creel-delivery-reed-tier1` · Task · enhancement, testkit, creel, reed · running · Darn-Write

## Seen

The Code-Review of `pr-session-mail-stand-ins` (round 1, finding F2) found that its new tier1 test reaches `creelfake.Delivery` through `startNoticeOrch`,
whose inventory verdict is "real", and that the halt row's unknown-strand refusal rests on the fake delivery's hand-written strand list.
The tree has no reed-backed creel delivery at tier1, so the parent accepted the use for that task.

## Idea

Give tier1 tests a creel delivery backed by a real reed, or by the recorded tmux kit, so a strand is known because reed holds it,
and move `startNoticeOrch` and its callers onto it.
