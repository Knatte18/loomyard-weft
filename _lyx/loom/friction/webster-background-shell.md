Master's wait counted a turn end past a background shell

These background shells ran past `background_shell_wait_min` (10 minutes) while Master's turn end was being judged, and may still be running in the session:

- exec </dev/null; cd /home/knatte/Code/loomyard-LYXHUB/test-suite-measure; until grep -q '^done$' .scratch-redundancy.log; do sleep 5; done; tail -5 .scratch-redundancy.log

