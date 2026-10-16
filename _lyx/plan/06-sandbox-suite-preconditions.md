# Card 6 — The sandbox suites name the shell the lyx check runs in

**Prosa:**
- `tools/sandbox/SANDBOX-REED-SUITE.md`
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`
- `tools/sandbox/SANDBOX-CREEL-SUITE.md`

**Intent:**
D8: each suite's `command -v lyx` pre-condition bullet names the shell it runs in; no other suite line changes but the creel suite's closing sentence.

- Reed suite: the bullet says the check runs in the pane shell of a strand whose command does not start a fresh shell, and that a strand whose command starts one (`--cmd bash`, a plain `--cmd claude`'s Bash tool) re-runs the operator's rc, so its bare `lyx` follows that rc's `PATH`, with `LYX_BIN` still naming the spawning binary.
- Shuttle suite: the bullet says `command -v lyx` run from a spawned agent's Bash tool names the spawning binary (`LYX_BIN`), even on a machine whose shell rc prepends another directory holding a `lyx`.
- Creel suite: the bullet says the check does not hold in this suite's interactive strands, the plain `--cmd claude` and `--cmd bash` strands, because each starts a fresh shell that re-runs the operator's rc; it points at the reed suite's bullet for the pane-shell guarantee and at `SANDBOX-SHUTTLE-SUITE.md` for the agent Bash tool check.
  The `LYX_BIN`-is-set bullet stays as written, and the closing sentence counts only the checks that remain: `LYX_BIN` being set is the live-substrate fact no `go test` can prove, and both remaining checks are pre-condition checks rather than assertions.

Every edited line keeps one sentence per line.
The creel suite's other lines belong to `creel-stage1-fixes`; this card changes only the pre-condition bullet and its closing sentence.

**Commit:** 6: Name the shell each sandbox suite's lyx check runs in
