# Writing the stand-in inventory with lookup subagents

One inventory subagent corrected its own report file with `perl -pi`, and the shell guard let it through.
The guard did refuse its later `sed -i`, with a message that names `perl -i`, so the `-pi` spelling seems to slip past.
