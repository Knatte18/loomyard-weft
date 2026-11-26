```yaml
lens: board-scope
outcome: reported
```

Task: board-scope lens on the sandbox-closed-area-build decision record and support log.

**Findings**

1. MEDIUM, consistency (Scope / D16): the record drops one of the entry's two cheap steps without saying so. The entry's "Before the area" asks the operator to start a hub's tmux server from a Konsole window rather than a VS Code terminal, because under the `vscode` label every agent of the hub may create user namespaces. D16 keeps only the README line. Neither Scope, the Rejected alternatives nor the Auto-mode assumptions keep, reject or reassign the Konsole step. Evidence: the entry body, "### Before the area: two cheap steps".

2. LOW, consistency (PATTERN changes, `PATTERN-config-layers`): the changed text conflicts with the one peer `config-keys-nested` prescribes for the same line. That peer (running, Plan-Write) rewrites the line to "…a meaning change ships a reinterpretation, a key move a move table, both run by `hubreconcile`; a per-task beam setting is read only via `dobbyengine`'s one lookup." This record's text is built on the old line, so whichever lands second would drop the other's clause. Fix: base the area clause on the peer's text, or name the merge for the implementer. Evidence: `config-keys-nested/_lym/discussion/decision-record.md`, its `## PATTERN changes`.

3. LOW, consistency (Open risks): the risk that `config-keys-nested` may rename reed's `pane_memory_max_mib` is settled. That peer's D2 says "Every other module keeps its keys as they are", and only loom and tapestry nest. Drop the line.

4. LOW, scope (Scope / Rejected alternatives): the entry's "What is not claimed" keeps Claude Code deny rules in shuttle's settings as "a cheap second line inside the area". The record neither includes nor rejects them. Evidence: the entry body, "## What is not claimed".

**Goal points of the entry**

- Seccomp connect supervisor over Landlock ABI 8 first: met (D1, D4).
- AppArmor per hub as the fallback: met (D15, D14).
- Decide what stays of the bwrap stages: met (Scope, D11, D13).
- Keep the strand pane memory cap under an area that refuses the bus: met (D9).
- Off by default; nothing starts when the area cannot, with no fallback to an unsandboxed start: met (D2, D3).
- Built in stages with a stop: met (D14).
- The area's own Claude config directory: met (D10).
- A fine-grained GitHub token, with the operator's `gh` login never entering: met (D11).
- Broker exits for deploy, window and hibernate, with `lym ide` refused: met (D13).
- README line on bypass permissions: met (D16).
- Konsole boot step: not met (finding 1).
- Cross-hub creel mail across an area: met (D8).
- Claude Code deny rules as a second line: not addressed (finding 4).
- Close the bus and other-hub tmux exposures: met (D4, D5).

**Entries, peers and parent commits checked**

- `reed-client-and-bus-guard` (T5, running Darn-Write): owns the shellguard bus rows, which the record leaves out. No conflict.
- `config-keys-nested` (T3, running Plan-Write): findings 2 and 3, otherwise no conflict.
- `dobby-run-watch` (T14): no record yet; no conflict seen.
- `creel-refusal-after-cd` (T6): its creel refusal overlaps D8 by package only. No conflict.
- `keep-landed-records` and the draft wave tasks: no conflict.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

The board holds no `Decision for sandbox-closed-area-build:` note, and the record has no `Added after Discussion` entries. There was none I could not reach.

**Tool calls**

- Listed the hub's worktrees and grepped the `reed-client-and-bus-guard` and `config-keys-nested` decision records. My context held only their board rows, not their records.
- Read `config-keys-nested`'s D2 and its PATTERN changes, and `reed-client-and-bus-guard`'s sections. I needed the exact peer texts to check for conflicts.
