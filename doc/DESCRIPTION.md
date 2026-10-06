OONI Probe, from the Open Observatory of Network Interference (a Tor Project
community), measures network interference: which websites and services are
blocked or reachable from a given connection. Published measurements form
OONI Explorer, the world's open dataset of censorship.

This package runs OONI Probe as a scheduled system service on your server: on
a timer it tests a bounded set of URLs from this network only, then publishes
anonymized results to OONI. It does not inspect anyone else's traffic, opens
no ports, and has no web UI. The default configuration runs the websites
suite twice a day with a URL cap; speed tests are disabled so background
traffic stays modest.
