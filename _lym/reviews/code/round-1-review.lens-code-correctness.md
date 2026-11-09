```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff, covering `decodeStoredAnswer`, `buttonAnswer` and the ask template in `runButtonVia`.

**List: function or branch, case run, result**

- **`decodeStoredAnswer`, kept-backslash branch.** Stored `\$A\$HOME`, a typed `$HOME` on tmux 3.4. The prefix `\$A` is cut. The regex matches `\$H` and the replacement `$$${1}` yields `$H`. Result `$HOME`. Correct. Verified against the real tmux 3.4 parser probe, which stored `\$A\$HOME`.
- **`decodeStoredAnswer`, typed backslash before a variable name.** Typed `\$HOME` is stored as `\$A\\$HOME`. The regex matches only the second backslash plus `$H`. Result `\$HOME`. Correct. The probe confirms the stored `\\$HOME`.
- **`decodeStoredAnswer`, typed `\\$HOME`.** Stored `\$A\\\$HOME`. The regex matches the last `\$H`. Result `\\$HOME`. Correct.
- **`decodeStoredAnswer`, non-name characters after `$`.** Tmux 3.4 stores `\$A$1`, `$-`, `$?`, `$$`, `$@`, `$#`, `$é` and `$}` with no kept backslash (probe). The regex does not match any of them, so the answer is unchanged. Correct. Typed `\$-` is stored as `\$A\$-`, which the regex does not match, so it stays `\$-`. Correct.
- **`decodeStoredAnswer`, doubled `$$`.** Typed `$$HOME` is stored as `\$A$\$HOME`. Result `$$HOME`. Correct. Typed `$$` is stored as `$$`. Correct.
- **`decodeStoredAnswer`, `${`.** Stored `\$A\${x}` decodes to `${x}`. Correct. The replacement `${1}` keeps the `{`.
- **`decodeStoredAnswer`, trailing `$` and empty answer.** Stored `\$Ax$` and bare `\$A` decode to `x$` and `""`. Correct. The empty case then takes `AskDefault` in `buttonAnswer`.
- **`decodeStoredAnswer`, marker ambiguity.** A 3.4 marker always begins with `\`, so `CutPrefix` of `\$A` fires. An old tmux stores `$A…`, so `TrimPrefix` runs. An old-tmux answer that starts with `\$A` is stored as `$A\$A`. It does not begin with `\$A`, so it is not misread. Correct.
- **`decodeStoredAnswer`, stale or empty option.** The ask template is the only writer of `@lym_button_answer` (grep). A stored value without the marker, such as `stale`, falls to `TrimPrefix` and is returned unchanged, as before. Correct.
- **`buttonAnswer`.** `TrimSpace(decode(raw))` followed by the empty check. Input `\$A  a b \n` gives `a b`. Whitespace after the marker is trimmed. A failed read still returns the same message. Correct.
- **`runButtonVia` ask template.** The Go raw string `"\` + `$A` + `%%%"` gives the template text `"\$A%%%"`. It parses to `\$A` plus the escaped answer (probe). The other metacharacters take the same path as before the diff, since only the `\$A` prefix is new. Correct. The probe did not exercise the real `command-prompt` `%%%` substitution, so I took the end-to-end case from the unchanged integration row. I did not run it.

**Findings:** none.

**Tool calls (2 beyond the manifest, both before the fork started):**
1. A tmux 3.4 parser probe via `source-file` on a private socket, because the manifest lacked the actual escape behaviour.
2. A grep for `@lym_button_answer` writers and readers, to confirm the template is its only writer.
