# Name config keys as <module>.<path> in Go comments

`config-key-comment-sweep` · Task · enhancement, docs · draft

After `config-keys-nested` nests the config keys by area, about 525 Go comment lines in about 199 files, plus single-word keys on top, still name a template key bare.
PATTERN-config-key-names asks that such a comment say `<module>.<path>`.

Sweep every Go comment and package doc that names a config key to its nested `<module>.<path>` form.
A comment-only change, no behaviour or user-visible text.
Deferred from T3 code review round 1, finding F1, to keep that run short.
