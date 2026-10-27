```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- The grep covered `internal`, `docs`, `PATTERN.md` and `pattern`. It found no remaining "dependency layers and Independent" list and no remaining old "Ready holds the open tasks" or "open task with no open dependency is readyLayer" wording.
- The `render.go:4` and `render.go:238` lists now match each other and the comment the review says was already edited at `render.go:317`.
- The `standingCell` comment lists Draft, which matches the `layerSection` fall-through the review describes.
- The `doc.go:176` sentence now says "open, released tasks". That agrees with the Draft sentence a few lines below it in the same paragraph and with `layer.go:29`.
- `docs/overview.md:359` is a twin that still lacks Draft. The reviewer rejected that finding with a reason, so I raise nothing.

Tool calls:
1. `grep` over `internal`, `docs`, `PATTERN.md` and `pattern` for the old wordings, to find twins my context did not show.
