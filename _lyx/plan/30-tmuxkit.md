# Card 30 — tmuxkit

**Create:**
- `internal/testkit/tmuxkit/tmuxkit.go`
- `internal/testkit/tmuxkit/tmuxkit_test.go`
- `internal/testkit/tmuxkit/tmuxkit_integration_test.go`
- `internal/testkit/tmuxkit/testmain_test.go`
- `cmd/lyx/tmuxkitspawn_test.go`

**Edit:**
- `internal/testkit/enforcement_test.go`
- `cmd/lyx/tierpurity_test.go`
- `CONSTRAINTS.md`
- `docs/overview.md`

**Uses:**
- `internal/testkit/scankit/scankit.go`

**Intent:** Add `internal/testkit/tmuxkit`, one isolated tmux socket directory per test package (D17), with the API named in the overview's `tmuxkit-api` Shared Decision.
`Main(m)` creates a short per-process directory with `os.MkdirTemp`, sets `TMUX_TMPDIR` to it, unsets `TMUX` and `TMUX_PANE`, runs `m.Run()`, then runs `tmux -S <socket> kill-server` for every socket under its own directory only and removes the directory, returning the run's exit code.
It fails setup, naming `TMPDIR` as the remedy, when the directory plus `tmux-<uid>/` plus reed's longest key (61 bytes) would exceed the 107-byte socket path limit.
When `tmux` is not on `PATH` it sets the environment and skips the sweep;
on Windows (psmux) it runs the tests unchanged.
`Socket(t, tmux)` returns a unique `-L` key and registers `kill-server` on that key in `t.Cleanup`.
The pure parts (path-length check, environment setup, socket listing of its own directory) are tested untagged in `tmuxkit_test.go` without spawning;
`tmuxkit_integration_test.go` proves against a real `tmux` that a server started on a `Socket` key or under `Main`'s directory is gone afterwards and that no socket lands in the default directory.
`testmain_test.go` runs the kit's own tests through `Main`.

What this admits, bounded per D17 and D19: `tmuxkit` is the second kit exempt from the `os/exec` ban, bounded to running the `tmux` binary against sockets under its own directory or its own fixture keys;
a test binary killed by a panic or timeout skips the sweep and leaves one `lyx`-prefixed temp directory with its servers, while `Socket`'s `t.Cleanup` still covers ordinary failures;
and unsetting `TMUX` means no test reaches the caller's own tmux server.

The Testkit scan exempts `tmuxkit` from the `os/exec` ban and bars every other kit's non-test files from importing `tmuxkit`, as it does `lyxbin`.
`tmuxkitspawn_test.go` defines a `tmuxkit` spawn once, parallel to `gitkitspawn_test.go`: every `tmuxkit` export except `Main`.
`tierpurity_test.go` bars such a reference from untagged files.
Prove both: an untagged `tmuxkit.Socket` call fails Tier Purity, and a kit importing `tmuxkit` fails the Testkit scan, each planted and removed, named in the commit body.
`CONSTRAINTS.md`: the Testkit Invariant gains the `tmuxkit` `os/exec` exemption with its bound and the rule that no other kit imports `tmuxkit`;
Test Tier Purity gains the `tmuxkit` rule beside its `gitkit` rule, naming `cmd/lyx/tmuxkitspawn_test.go`.
`docs/overview.md`'s Tests passage names `tmuxkit`.
No package adopts `Main` in this card.

**ImpactSummary:** Two invariant scans and two `CONSTRAINTS.md` sections gain the new kit's rules; no existing test changes behaviour.
