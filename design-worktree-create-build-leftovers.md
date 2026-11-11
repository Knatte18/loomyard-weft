# Worktree-Create halts on a reused slug path holding only obj/ leftovers

`worktree-create-build-leftovers` · Task · bug, batten, fabric

Depends on: [`push-notice-anchor-dir`](design-push-notice-anchor-dir.md)

Imported from [issue #108](https://github.com/Knatte18/loomyard/issues/108).

Seen: `lym beam merge-parent support-for-dop` halted its batten run at Worktree-Create because the directory `<hub>/support-for-dop-merge-in` already existed and was not a worktree.
It held only `src/csharp/NORCE.Models/*/obj/Debug/...` build leftovers (15 files, generated AssemblyInfo and editorconfig) dated from the previous merge-in run of the same slug a day earlier, which had landed and been torn down.
Something (an IDE or a build triggered after teardown) recreated obj directories under the removed worktree path, and Worktree-Teardown does not check for or clean such leftovers.

Workaround: inspected the directory, removed it by hand (`rm -r` on the literal path), and ran `lym batten run support-for-dop-merge-in` again.

Expected: a reused slug whose old path holds only build output (obj/bin) is not blocked; Worktree-Create, or Worktree-Teardown of the earlier run, removes a directory that contains nothing but build artefacts, or the halt names it as such so the parent does not need to inspect it.
