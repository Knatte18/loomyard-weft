# Comment lint: bound a // comment line to 120 characters of text

`comment-lint-line-length` · Note · enhancement, loom

## Seen

`lym loom lint-comments` (`internal/commentlint`) checks only where a `//` comment line breaks.
Nothing bounds a comment line's length, so a one-sentence-per-line comment can run very long.
Requested by the `comment-cleanup` writer, which reports the operator approved it (2026-11-07).

## Wanted

- A `//` comment line the diff adds or changes holds at most 120 characters of text after the `//`; indentation is not counted.
- A line whose URL, code span or identifier alone passes the limit is exempt.
- A finding fails the gate as the existing break findings do.

The same rule goes into `scribe:prose`, in the Scribe hub.
