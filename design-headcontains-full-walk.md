# HeadContains walks the whole history on every source-ahead seed pass

`headcontains-full-walk` · Note · enhancement, stencil, fabric

## Seen

The PR-Gate review of `binary-change-and-rules` (PR #469) found that `gitrepo.HeadContains` (`internal/gitrepo/blobread.go`) walks HEAD's whole history through go-git whenever a seed pass finds a source-ahead stencil.
In a long-history repository that cost lands on every command until the pair syncs with main.

## Idea

Answer the ancestry question with a merge-base or a walk bounded by commit date or depth instead of a full history walk.
