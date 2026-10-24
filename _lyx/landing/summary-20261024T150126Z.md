# Restore to PATTERN the moved rules that bind code outside their package

The PATTERN cut in `04aafa88c` moved package-local rules out of `PATTERN.md` into their owning package's doc.
`PATTERN.md` is inlined in every spawned agent's prompt, but a package doc is read only by an agent already working in that package.
Six of the moved rules bind code outside their package, and no test catches a break, so they return to the overview.

## What changes

- **Six overview entries restored**, each one line with no background file:
  - `PATTERN-sparse-hub-base`, under CLI and config: any writer of a hub base file keeps it sparse.
  - `PATTERN-parent-lock`, under Fabric and git: a local parent branch moves only in Finalize, under its parent lock.
  - `PATTERN-transient-stop` and `PATTERN-friction-capture`, under Shed and loom: the transient mark's one owner and boundaries, and the Go-written friction note on a loom halt, crash-resume or webster refusal.
  - `PATTERN-agent-name` and `PATTERN-wait-mark-display-only`, under Agents and prompts: names go through `internal/agentname`, and no Go decision reads the wait-mark pane options.
- **Package docs cite the IDs.**
  The rule's sentence in `internal/hubreconcile`, `internal/parentlock`, `internal/agentname`, `internal/reedengine`, `internal/shedengine` and `internal/friction` now names its restored entry.
- **Voucher clause restored.**
  `internal/friction/doc.go` again states that `lyx loom start` vouches for the resume it spawns, so a deliberate resume leaves no crash-resume note.
  The cut had left that clause only in a comment in `internal/loomcli/start.go`.

The overview grows from about 7.0 KB to about 8.1 KB, well under the headroom line.

## Design decisions

- `transient-stop` and `friction-capture`, which the audit only leaned towards restoring, come back too:
  no scan covers a mark site or a halt or refusal path outside the owning package, so they meet the same test as the other four.
- Each entry names the package whose doc holds the detail instead of linking a background file.
  The format checker admits only a `pattern/` link, and a background file would hold a second copy of the doc's rule.
- Each entry keeps the `(test)` marking its removed line carried, so only `friction-capture` has one.
- No tripwire test is added.
  One would let a rule leave the overview later, but each needs its own scan.
- `hub-wide-config`, `recipe-owns-failures` and `batcher-registry` stay in their package docs, as the audit judged.

## To check by hand

- Read each restored line against its package doc section for a clause the line contradicts.
