# Support log — wait-world-lock-grace-flake

## Interview

A darn run holds no interview; the writer read the board entry and the code and made its calls alone.

## Rejected alternatives

- Keeping `LoopSpec.LockGrace` as a test seam: no caller sets it, and keeping it lets a fixture reintroduce the short grace.
- Widening every short timing in the test suite: bounds whose expiry is asserted, and "still blocked after X" checks, are deterministic in the failing direction, so changing them buys nothing.
- Moving `TestVerify_SlotGate` onto a fake clock: `Verify` takes a real timeout for its command, and injecting a clock there is a production change beyond this task.
- Raising the verifytree timeout to several seconds: the slot hold scales with it, so each second costs about two seconds of suite time.

## Review rounds

_No rounds yet._

## Question ledger

- Is the wait-world fixture already fixed? Resolved: yes, `743c11f0d` removed its `LockGrace`; this task removes the orphaned field.
- Do the `lock` unit test and the githubclient test count as the integration tier? Self-pick: yes, the integration run executes every test in the module.
- Is a timeout on a success path in scope beside a lock grace? Self-pick: yes, it makes verify nondeterministic the same way.
