# config: bare lyx config <module> from an agent writes an empty layer file

`config-bare-empty-write` · Note · bug

Imported from [issue #632](https://github.com/Knatte18/loomyard/issues/632).

Seen (2026-10-15): a Discussion-Write agent in pair webster-pack-end-of-prompt ran 'lyx config webster' with no arguments, expecting a read. It replied 'edited and synced _lyx/config/webster.yaml' and left a 0-byte _lyx/config/webster.yaml in the pair's worktree layer (behind the weft symlink, so git status shows nothing). The bare form opens an editor; with no interactive terminal the editor exits at once and lyx saves the empty buffer as the layer. Workaround: the orch never runs lyx config without --set or --print. Expected: with no TTY, a bare 'lyx config <module>' refuses (or prints) and never writes; an empty editor result is never saved over a layer.