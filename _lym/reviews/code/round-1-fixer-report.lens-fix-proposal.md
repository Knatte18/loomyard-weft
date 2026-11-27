```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3, all of which the fixer applied as proposed.

**LOW: "every request outside the main conversation … background and helper requests" is slightly too broad (F1 proposal)**
- **Changed text:** the `subagentPromptCacheTTLEnvKey` doc ("every request outside the main conversation: subagents, forks, and background and helper requests such as compaction"). The same claim is in the `subagentPromptCacheTTL` doc, `doc.go` ("helper requests"), `docs/overview.md` and the summary's Cost bullet.
- **Evidence:** the 2.1.332 bundle has `fbt=["repl_main_thread*","sdk","auto_mode","memdir_relevance"]`, and `UVn` picks `CLAUDE_CODE_PROMPT_CACHE_TTL` for those sources and the subagent variable only for the rest. So the auto-mode classifier and memory-relevance requests, which a reader would count as helper requests, take the role's TTL and not the hour.
- **Weight:** marginal. Claude Code's own settings text says "everything outside the main conversation … background and helper requests", so the prose is right if "main conversation" is read as Claude Code classes it. Saying nothing more is defensible. If a change is wanted, one clause in the constant's doc, saying Claude Code counts a few of its own helper requests as main conversation, is enough.

F2 and F3: I found no case where the proposal is wrong or breaks something elsewhere. For F2, `UVn` confirms that the main variable governs only the listed main-thread sources and that `FORCE_PROMPT_CACHING_5M` outranks it.

Tool calls:
1. `grep` of the Claude Code bundle for the source list and `UVn`; my context had only the review's description of them. The context-window pattern errored.
2. A second `grep`, fixed-string with byte offset, to locate the list after that error.
3. `dd` of the bytes at that offset, to read `UVn` and the list, which my context lacked.
