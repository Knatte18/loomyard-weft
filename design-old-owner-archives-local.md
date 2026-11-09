# The old owner pushes its unpushed tips to origin when another machine adopts

`old-owner-archives-local` · Note · enhancement, dobby, fabric

## Seen

On 2026-11-09 the operator noted that the adopting machine cannot know what the old owner holds locally: commits the old owner never pushed are invisible to adopt and lost to the new owner.

## Idea

When the old owner sees that another machine owns a beam or root, its dobby pushes every local tip origin lacks, under that subject, to an archive branch on origin, then stops its runs there.
The new owner sees the archive branches and decides what to bring in.