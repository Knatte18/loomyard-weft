```yaml
lens: fix-affected-set
outcome: reported
```

One LOW: F2 set not fully covered. internal/battenshed/deps.go:225 ("A nil Activity resolves to reporting no live run and no live wait") and internal/battenshed/innerrun.go:250 carry the stale phrase. F1 callers complete: loomcli/decision.go:856 and loomcli/looparm.go:169 carry the skip and are tested; battencli/wire.go:605 reads waiterless runs on purpose.

Tool calls (1): one grep over non-test Go files for ReadAgentActivity callers, WaiterGone uses and live run phrasings; the context lacked the full sets.
