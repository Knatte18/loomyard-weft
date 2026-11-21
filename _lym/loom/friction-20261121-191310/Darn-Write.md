# Trimming every doc.go over many Sonnet subagents

A first-pass brief that said "when unsure whether a sentence is a contract, keep it" made almost every Sonnet subagent cut only 5–15 %.
A whole second fan-out was needed with a section-by-section method and a rough target before files shrank by half or more.
The subagent concurrency cap of 20 also left part of the first wave queued behind the slow large files.

`lym loom lint-comments --base` checks committed HEAD, not the working tree, so a fix could be checked only after a commit.
Running `tools/godocreflow` over the trimmed files to clear the 624 wrapped-break findings made things worse (916 findings):
it skips whole comment blocks that hold a heading or list and joins lines past the length limit.
That commit was dropped, and the findings were fixed by hand-editing subagents instead.
