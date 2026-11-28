Diagnosing a gate timeout that a hibernation caused

The tier1 timeout counts wall-clock time across a machine hibernation.
The machine slept from 2026-11-27 22:47:54 to 2026-11-28 08:52:17 while tier1 was running, and on wake the gate killed tier1 as past its 30-minute limit.
The report asks the reader to find a hang or raise `gate.tiers.tier1.timeout_min`, and neither applies; only the system journal showed the cause.
