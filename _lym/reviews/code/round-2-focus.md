---
round: 2
focus:
  - "In contracts/orchrole/role-slack.md, check whether each bullet that grants authority or relays mail names the sender `<shortname>:SLACK`, and whether any mail that merely opens `[slack]` from another sender could read as an operator instruction. Compare against the role file as it stood at b30cccce."
  - "In the orch role and its on-demand files under contracts/orchrole, check that every binding rule of the role at b30cccce survives with the same force, in particular every rule that conditions authority on a sender."
  - "In internal/orchengine and internal/orchcli tests changed by the diff, check whether any new assertion pins static role prose instead of a render behaviour such as the filled `Decision for <slug>:` marker."
  - "In docs/overview.md near the `lym help index` sentence, check that the text matches what the orch role file now carries."
---

Focus targets where the round 1 review found defects, stated without reference to its findings.
