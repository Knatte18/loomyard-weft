# Fewer serial top-level tests in internal/fabricengine

`fabric-serial-phase-tests` · Note · enhancement, fabric, testkit

## Seen (orch, 2026-11-09)

`docs/research/speed/fabricengine-shards-2026-11-09/report.md`, measured on OSL-1046 under its normal load, once per case: 46 of the package's 400 top-level tests are not marked parallel, and the serial chain they form is 143 s, 59.5 percent of the 242 s one-process wall.
Inside that phase the package keeps about one core busy while the machine is two-thirds idle, and shards did not move the wall.

## Idea

Find why each of the 46 is serial (environment variables, a shared hub, a missing `t.Parallel()`) and make as many parallel as their cause allows.
The report gives an upper bound of about half the wall.
The wall figure is from OSL-1046, where Cortex XDR slows every new binary and file; it may not hold on another machine.
