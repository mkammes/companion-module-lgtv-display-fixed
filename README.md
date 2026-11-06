# KCM build: 2.1.0-kcm.6

Based on upstream `bitfocus/companion-module-lgtv-display` **v2.1.0-beta.2**, which already
includes the three fixes from this fork's 2.0.1-kcm.1 (Send Key values, direct Wake-on-LAN,
30 s auto-reconnect). On top of that, for LG TVs on Wi-Fi (tested target: SK8000PUA, webOS 4):

1. **No more stale "on".** When a TV drops off the network the socket stays open and every
   read times out, and the module kept the last values forever. Now 2 failed polls in a row
   close the connection, reset state to unknown and start the reconnect loop.
2. **Wake-on-LAN burst.** Power On sends 5 rounds, 0.5 s apart, to every known MAC (config
   MAC, new optional *Second MAC*, and the MACs the TV reported), to the configured broadcast,
   255.255.255.255 and the TV's own IP, on ports 9 and 7.
3. **MAC auto-detect.** On connect the module asks the TV for its Wi-Fi and wired MACs
   (`GET_MACADDRESS`), exposes them as `mac_wifi` / `mac_wired`, and logs a warning when
   neither matches the config. The hidden 82888 menu often shows the wired MAC even when the
   TV is on Wi-Fi.
4. **Fast reconnect after Power On**: retries every 3 s for 2 minutes instead of 30 s.
5. New variable `connected`.
6. **kcm.3: no stalled reconnects.** A disconnect while a read was in flight could leave
   that read hanging forever (the library drops the socket's listeners), which blocked the
   command queue and stopped the reconnect loop. Now every queued task times out after 25 s,
   the queue is replaced on disconnect, results from superseded connect attempts are
   ignored, and a 10 s watchdog restarts a stalled reconnect loop or drops a connection
   whose polls stopped succeeding for 30 s.
7. **kcm.4: visible command errors.** A failed command logs a warning with the TV's reply and
   sets the `last_error` variable.
8. **kcm.5: faster off feedback.** After the TV acknowledges Power Off it shows as off at once,
   and a poll whose power read gets no answer skips its other reads (5 s instead of 20 s).
9. **kcm.6: wake until it answers, and a power_pending variable.** Power On repeats the
   Wake-on-LAN burst every 3 s for up to 60 s until the TV reports on (fixes Off then On within a
   few seconds, and TVs slow to rejoin Wi-Fi). New `power_pending` variable: `turning_on`,
   `turning_off`, `no_response` (on: no answer in 60 s; off: TV not connected or command failed),
   empty once the TV answers. Power Off no longer throws when the TV isn't connected.

Module id stays `lgtv-display`, so it can replace the stock version on existing connections.
Build: `npm install && npx tsc -p tsconfig.build.json && npx companion-module-build`.

---

# companion-module-lg-display

- See [HELP](./companion/HELP.md) for getting started information
- See [CHANGELOG.md](./companion/CHANGELOG.md) for release notes
- [LICENSE](./LICENSE)
- This module uses the [lgtv-ip-control library](https://github.com/WesSouza/lgtv-ip-control)
