# lyx report timing counts sleep as active time when the system journal is unreadable

`report-timing-unreadable-journal` · Note · bug, report

Imported from [issue #803](https://github.com/Knatte18/loomyard/issues/803).

## Seen (orch, 2026-10-21, OSL-1046)

`lyx report timing` takes machine sleep windows from the system journal.
On this machine `journalctl` cannot read the system journal, because the AD user is in neither `adm` nor `systemd-journal`.
The report then subtracts no sleep, so sleep and quota waits are counted as active run time, and it says nothing about it.

## Workaround

None; the numbers from this machine overstate active time.

## Expected

When the journal is unreadable, the report finds idle time another way, such as recorded tender pauses, hub-wide idle windows, or gaps in the runs' own timestamps.
It also states in its output that sleep could not be read and the numbers may include it.
