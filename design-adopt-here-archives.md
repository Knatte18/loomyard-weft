# lym adopt --here behind the lym s shortcut, and lym adopt --archives lists the old owner's archives

`adopt-here-archives` · Task · feature, batten, tender, dobby, orch, fabric, shortcut · draft

Decision record: [Sketch](decision-adopt-here-archives.md)

Depends on: [`adopt-owner-handover`](design-adopt-owner-handover.md)

Filed from the Sketch `adopt-robust`.

Takes: adopt-worker-record's worker, record, follow and resume; adopt-owner-handover's archive-owner tag name function
Delivers: lym adopt --here with its owned_here outcome and record take-up; the s shortcut as adopt --here --no-orch, beam start, reed attach; lym adopt --archives over a fabric read of archive-owner tags against each repository's origin branch
Exports: the fabric archive-owner tag read

The part changes these packages:

- internal/adoptcli (about 450 changed lines)
- internal/shortcutcli (about 10 changed lines)
- internal/fabricengine (about 150 changed lines)
