# Support log — darn's review hard cap defaults to 2

## Interview

A darn run holds no interview; the writer read the board entry and the code and made its own calls.

## Rejected alternatives

- Dropping the `light` profile's darn values since they equal the new defaults: rejected, because the board entry asks for them and they keep the profile's bounds should a hub raise its darn defaults.
- Removing the hub's `review_hard_max_bounces: 2` from `_board/_lyx/config/darn.yaml` in this run: rejected, because that file is hub config on Bolt, outside the task worktree, written only from the prime.

## Review rounds

_No rounds yet._

## Question ledger

- Should the light profile carry darn values that equal the new defaults? Self-picked: yes, as the board entry states.
- Who removes the hub's own darn.yaml value? Self-picked: the parent, from the prime, after landing; the writer names it in the change description.
