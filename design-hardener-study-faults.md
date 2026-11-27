# Faults the hardener study read in fabric, cleanup and teardown code

`hardener-study-faults` · Task · bug, fabric, cleanup, batten · halted · Darn-Write

## Seen

`docs/research/hardener-works-study.md` (`c6618b538`), section "Faults found by this study", read these in the code; none was driven:

- a refused credential during a one-repository seed keeps the hub and says origin may still take the push;
- an empty teardown job clears an earlier teardown's pending record;
- the `cleanup` remote sweep counts local tags as landed and never pushes them;
- `--reset` is advised without regard to unpushed work;
- `fetchRunForCatchUp` reads no records layout;
- a `lym fabric remove` pending record can outlive its pair (`cd8090733` fixed only the cancel's records, by choice).

## Idea

Confirm each against the code with a test that fails first, and fix the confirmed ones.
