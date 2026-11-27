```yaml
lens: fix-consistency
outcome: reported
```

Task: check the round-2 fix set for agreement with the sibling text it touches (fix-consistency lens).

- **LOW**: the cross-reference in D1 now points at the wrong section. D1 still says "The child's first query script runs the post-launch per-desktop tiling probe (Constraints)." The fix deleted the "Not yet observed" list from Constraints, and that list held the only description of the probe ("passes when `rootTile` returns two different root tiles for one output and two desktops"). The description now sits only in the new Open risks bullet, so D1's pointer to Constraints leads nowhere.

- **LOW**: the homeless-fallback rule was added to one D8 bound but not its twin. The fix changed the first D8 bound to "anchors that hub's run windows on the current desktop, homeless until their next placement." The bound right after it still ends "the child then anchors on the current desktop" and says nothing about the child taking no home. It is the same fallback, so the two bounds now describe it differently.

- **LOW**: the Review criterion about the help text doesn't mention the new SIGTERM exception. D12 now says the child runs `window_command` "on every exit path but SIGTERM." The Review criterion still asks that the help and template comments say "under placement the placement child runs `window_command`, with no wait," with no word of the exception, so the text that criterion asks for would overstate the guarantee.

No further tool calls made.
