```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set against its surroundings (lens fix-consistency).

no findings

- I grepped for `harden-live-copies` and `focusPath` across `.go`, `.md` and `.yaml` files. The only `focusPath` hits left are parameter and local names in `internal/burlerengine/prompt.go` and `internal/shedadapters/bouncer.go`, not references to the renamed builder.
- No other file, comment or doc outside the new rewrite names `harden-live-copies`.
- The `template.yaml` comment and the `hardenshed` doc already agree with copies running under `harden-live`.
- The summary's design-decision bullet, the "No hub gate writes" bullet and the Merge danger section agree with the diff.

Tool calls:
1. `grep -rn "harden-live-copies\|focusPath\b"` over `.go`, `.md` and `.yaml`, excluding `_lym/reviews`, `.scratch`, `.lym` and the new rewrite files. I ran it because my context held no repo-wide view of leftover references to either retired name.
