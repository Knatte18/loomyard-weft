# twill nested-spawn check counts a parent's spawns inherited into a fork's transcript, halting the run

`twill-nested-spawn-inherited` · Task · bug, burler · running · Code-Burler-Review

Imported from [issue #276](https://github.com/Knatte18/loomyard/issues/276).

## What happened

A run halted `failed` at Plan-Burler-Fix (`anomaly: producer-hard-failure`), with `<home>` replacing the home path:

```
shedadapters: Plan-Burler-Fix (burler-fix): round 2 attempt 2: fix: burler: twill fixer fork "<home>/.claude/projects/<project>/<session>/subagents/agent-abc5528dbce58b86c.jsonl" attempted 4 nested spawn call(s) — forks cannot nest and must never spawn a fork or subagent, even when the attempt was denied; no target change was found: no fork wrote a file or ran a state-changing git command outside a repository it created; way forward: resume the run
```

The run was resumed and went on to Finalize.

## Why it is abnormal

The subagent transcripts show that the 4 calls lym counted are not the named fork's own.

| transcript | agentType | spawnDepth | parent | `Agent` tool_use ids in the transcript |
|---|---|---|---|---|
| `a896671709490d648` | general-purpose | 1 | none | `01MaMK`, `018Bmv`, `01Nmn3`, `01QQvx`, `01Xri2` |
| `a9cd9fb6e1c651e3f` | fork | 2 | `a896...` | `01MaMK` |
| `a9306719cc045d665` | fork | 2 | `a896...` | `018Bmv` |
| `afa081b3c803fd519` | fork | 2 | `a896...` | `01Nmn3` |
| `a490a283475e1c95c` | fork | 2 | `a896...` | `01QQvx` |
| `abc5528dbce58b86c` (named in the halt) | fork | 2 | `a896...` | `01MaMK`, `018Bmv`, `01Nmn3`, `01QQvx`, `01Xri2` |

- `abc5528...` was spawned by call `01Xri2` (its `meta.json` `toolUseId`), the last of the five.
- Its other four ids are its parent's earlier sibling spawns, carried over in the inherited history and stamped with the child's `agentId`.
- Their timestamps equal the parent's calls and precede the `01Xri2` call.
- Each of the four returned `Async agent launched successfully`, so none was denied.
- The count of 4 is the five inherited ids minus the fork's own spawning call.

So the halt blames a fork for spawns its non-fork parent made.

## What to do

- In the nested-spawn check, leave out tool_use ids that also appear in the parent transcript: they are inherited history, not the fork's attempts.
  A fixture needs a subagent that spawns several forks in turn, since only the later forks inherit earlier spawn calls.
- Decide separately whether a general-purpose subagent of a fixer may spawn forks.
  If it may not, the refusal should name that subagent, and the spawn guard should deny the call when it is made.

Provenance: task slug `fabric-core-foundation`, from note `loom-halt.md`.
