# Diagnosing the shuttle idle-check miss without its evidence

The task said to check the idle reading against `startup-capture.txt` in two run directories, `.lym/shuttle/f31a8e…` and `.lym/shuttle/6c6752…`.
Neither directory exists anywhere on this machine: the worktree of the source run (`inmemory-store-windows-root`) is gone, and no hub log holds the failure.
The file would not have helped anyway, because it holds the startup probe's boot screen, not the screen at the failed prompt delivery.
A lot of time went into looking for the evidence and into reading the box-detection code for a layout mismatch that could not be confirmed.

Issue #136 lost its body, so the second part had only a title to work from.
A board entry imported from an issue would be easier to act on if the import copied the evidence it names (captures, log lines) into the entry.
