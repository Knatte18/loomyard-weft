# Command-help byte-cap test is not run by card or batch gates

`cli-form-test-in-card-gate` · Task · bug, gate, cli · starting · Seed-Child

Depends on: [`memory-guard-infra-stop`](design-memory-guard-infra-stop.md)

Imported from [issue #274](https://github.com/Knatte18/loomyard/issues/274).

**What happened.**
A card added text to the `lym board upsert` help and pushed it past the 2000-byte cap that `TestCLIForm_EveryCommand` in `cmd/lym` holds.
Neither that batch's card gates nor its batch gate ran the test.
It first failed at a later batch's `--main-packages` gate, and was fixed there.

**Why it is abnormal.**
A change passed every gate of its own batch while breaking a test.
The failure landed on a different implementer, whose cards had not caused it.

**What to do.**
Make a scoped gate run this test when command help changes, for example by marking it `//lym:guard`
(its own doc comment says its checks are pure and run no invocation),
or by having the go scoper include `cmd/lym` when a package that registers commands changes.

Provenance: task `dobby-run-watch-build`, note `04-run-watch-wiring.md`.
