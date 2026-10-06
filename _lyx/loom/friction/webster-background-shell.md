Master's wait counted a turn end past a background shell

These background shells ran past `background_shell_wait_min` (10 minutes) while Master's turn end was being judged, and may still be running in the session:

- mkdir -p .scratch/after && cat > /tmp/claude-1000/measure.sh <<'EOF'
cd /home/knatte/Code/loomyard-LYXHUB/test-suite-prune-core
for i in 1 2 3; do
  cat /proc/loadavg > .scratch/after/load-timing$i.txt
  go run ./cmd/testtiming -tags integration,tmux > .scratch/after/timing$i.txt 2>&1
done
cat /proc/loadavg > .scratch/after/load-cover.txt
for p in fabricengine websterengine reedengine shuttleengine loomcli shedadapters; do
  go test -tags integration,tmux -cover -count=1 -coverprofile=.scratch/after/cover-$p.out ./internal/$p > .scratch/after/cover-$p.txt 2>&1
done
echo MEASURE_DONE
EOF
bash /tmp/claude-1000/measure.sh > /tmp/claude-1000/measure.log 2>&1

