```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the change set against its surroundings.

1. **LOW**: `pattern/PATTERN-test-speed.md`, the new "It builds only when …" list. The clause "a targeted directory or test file cannot be read" leaves out the unparseable case, so the "only when" list is narrower than the code, which is the defect F5 raised.
   - Evidence: `impactset.TestsUseLymBinary`'s doc says "A directory or test file that cannot be read or parsed counts as using it", and the code returns true on a `parser.ParseFile` error.
   - Change: "cannot be read or parsed".

I found no other disagreement. The twins I checked agree: the `gatecli` package doc, the `tierrun.go` and `copies.go` docs, `PrebuiltLym`, `PrebuiltLymEnv` and `running-tests.md`. The config template only lists the `prebuilt_lym` key name, and nothing references the old copies test name.

Tool calls:
- `git grep` for build-condition wording: looking for twins in files not in context. The output was too large to be shown.
- A narrower `git grep` of the same: the first output was unusable.
- `grep`/`git grep` for the template's `prebuilt_lym` text and the old test name: not in context. The command failed on a regex limit.
- A retry of that: it returned the line number and the old-name search (no references), but no template text.
- `sed -n` on template line 10, read only: to get the `prebuilt_lym` sentence.
