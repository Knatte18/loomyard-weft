```yaml
lens: fix-consistency
outcome: reported
```

One LOW: internal/battenshed/deps.go:225 and internal/battenshed/innerrun.go:250 still say "no live run" for the nil Activity case, which now reads "no run". notice.go:53 and notice.go:56 stay correct; battencli/parentlock.go:98 is a different seam.

Tool calls (1): grep for live run phrasings over internal/battenshed and internal/battencli; the context lacked a search for twin phrasings.
