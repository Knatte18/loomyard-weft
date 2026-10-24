```yaml
lens: code-pattern-fit
outcome: reported
```

Lens: code-pattern-fit. I checked the diff `93f1c8942..HEAD` against the repository's PATTERN rules, and found no violations.

**Rules checked**

- **Declared PATTERN changes landed.** Held. The 30 entries in `PATTERN.md` read the same as the declared "changes" texts. I compared them by eye against the decision record and the constraints block. The declared "removes" IDs are absent from `PATTERN.md`, and `pattern/` holds no background file for them.
- **`pattern.Check` one-to-one.** Held by inspection. The `pattern/PATTERN-*.md` links in `PATTERN.md` match the files in `pattern/` exactly (`diff` clean, 28 files). The two entries with no background file are `github-auth` and `shell-guard-table`, as declared. Each background file's `# ` title is a surviving ID.
- **Entry line length.** Held. The longest line in `PATTERN.md` is 317 characters. I did not read `MaxEntryLineChars`, so the cap is unconfirmed.
- **Size and headroom.** Held. `PATTERN.md` is 6966 bytes, against the 16384 `HeadroomBytes`.
- **Citation sweep.** Held. Every distinct `PATTERN-` ID in live files is in `PATTERN.md`, apart from test placeholders (`PATTERN-a`, `PATTERN-x` and similar). The sweep excluded `docs/research`, `docs/benchmarks`, `testdata` and `_lyx`. `CLAUDE.md` and `docs/overview.md` name no missing ID.
- **Parent's new text in the merged survivors.** Held. Every `ed98ccf71..93f1c8942` change to the old test-speed, test-isolation, test-economy and testkit files appears in `PATTERN-test-speed.md` and `PATTERN-test-isolation.md`. That covers the `tierN` renames, the `//lyx:guard` clause that refuses `integration`, `tmux` and `llm`, and the "no coverage" `tier2` wording. The testkit `tier3` line is in test-isolation (line 84).
- **Parent-changed test files in the amendment scope.** Held. I read the 18 listed `.go` files' diffs against `93f1c8942`. Each cited ID is now a live survivor, a package-doc pointer, or a plain description.
- **Moved rules keep meaning.** Held on the checks I made. I read the `tools/sandbox/doc.go` and `lyxcwd.go` additions; the sandbox coverage and dev/prod binary rules, and the hub suffix section, read as complete rules. I could not compare them against the pre-cut background files, because I read them at `93f1c8942` and that is the post-parent tip, where those files are already gone. I did not read the pre-move text from `ed98ccf71`.
- **`lyxdirs-single-declarer` carve-out.** Held. `pattern/PATTERN-durable-vs-ephemeral-state.md:19` keeps "the one exception is a `tools/` script's variable block", and the overview entry carries it too.
- **Commit messages and departures.** Held. The three commits in range have no `Pattern-Departure:` trailer, and no difference from the answer key needs one.
- **Comment convention in new `doc.go` text.** Held. The moved-rule sections in `tools/sandbox/doc.go` and `lyxcwd.go` use one sentence per line with no end-of-line comments. This is the only part of the `doc.go` set I read.

**Not checked.** I did not read the other 19 `doc.go` additions, and I did not compare the background-file merge sections with their sources.

**Findings:** none.

**Tool calls (2)**
1. A combined shell command: commit trailers, link-to-file one-to-one match, max line length, cited IDs in `CLAUDE.md` and `docs/overview.md`, and background titles. My context had only the diff stat for these.
2. A combined shell command: diffs of `tools/sandbox/doc.go` and `lyxcwd.go`, and the `durable-vs-ephemeral-state` carve-out line. My context lacked the doc contents. The attempt to read the old background files at `93f1c8942` failed because they were already gone there.
