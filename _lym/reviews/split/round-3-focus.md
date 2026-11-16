---
round: 3
focus:
  - "Criteria placement: for each part in _lym/raddle/proposal.yaml, check that every criterion bullet tests only behavior the part's own contract, or a part on its depends_on path, delivers. Settle whether any bullet asks for a hold write, a takeover-entry production writer with its rejected-push retry, or a dobby pass removing a local pair, in a part that runs before the part delivering it."
  - "Fix-tries reader route: in machine-registry-holds-catchup and adopt-foreground, check that both bodies name the same package and function for the fix-tries.json count reader the upgrade's first read calls, and where that first read runs. Settle whether the route forms an import cycle, given that internal/adoptcli imports internal/dobbyengine and internal/pauserecord."
  - "Moved criteria: check that each criterion moved between parts still has its decision id built by a part it lands in, and that the receiving part has the delivering part on its depends_on path."
---

Focus targets the seams the round 2 fixes touch: criteria placement and the reader's interface.
