```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checked the claims in F1's added D4 bullet and criterion line.

no findings

- "Every other value the page renders from `window.BOARD` is inserted as text": this is a design commitment about the page, not a claim about existing code. The values it names match what D1 projects and what D4 renders: header render time and owner, project and entry fields, the facts line, section names, and design and decision file names.
- "A design or decision file name enters an `href` only as a relative path opening with `./`": this holds for the names the code produces. They take the form `&lt;prefix&gt;&lt;slug&gt;.md`, built from `designFilePrefix` and `decisionFilePrefix` in `internal/boardengine/replay.go:34-36`. The design prefix comes from `board.yaml` and is configurable (`internal/boardengine/config.go:206-210`). Because the path always opens with `./`, even a configured prefix holding a colon cannot become a URL scheme, so the claim holds.
- The new criterion line ("renders a title or brief holding an HTML tag as literal text") matches the new bullet.

Calls:
- grep of `internal/boardengine` for where design and decision file names are built: I needed to know whether a file name could carry a scheme, and my context did not have the naming code.
