# Read-only classifier may admit a git reader flag that writes

`fabric-readonly-git-write-flags` · Note · bug, fabric, webster

## Seen

`webster-replan-state` landed (5a8be0274) with Webster-Review finding `d8-reader-write-flags-unbounded` (design, BLOCKING) open: the reader allow-list in `internal/fabricengine/refscanner.go` admits git reader commands without bounding their flags, so a reader given a writing flag could pass as read-only.

## Idea

Check the classifier against the flags each admitted git reader accepts, and reject any flag that writes.