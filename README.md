# OONI Probe — YunoHost package

Run [OONI Probe](https://ooni.org) as a system service that measures your own
network's reachability on a schedule (which websites are blocked from this
connection) and publishes anonymized measurements to OONI's open censorship
dataset. It does not inspect anyone else's traffic and has no web UI.

## Requirements

- YunoHost ≥ 12
- No Docker — installs the official static `ooniprobe` binary (checksum-pinned)
  under a dedicated system user.

## Install

```bash
yunohost app install https://path/to/this/repo
```

Options:

- `run_interval_hours` — how often measurements run (default 12).
- `url_limit` — max URLs tested per run (default 20; 0 = unlimited, not
  recommended).

Only the websites suite runs; speed tests are disabled by the pre-seeded config.

## Operate

```bash
systemctl list-timers | grep ooni          # next run
journalctl -u ooni-probe.service -n 50     # last run log
sudo -u ooni-probe HOME=/var/www/ooni-probe /var/www/ooni-probe/ooniprobe run websites   # force a run
sudo -u ooni-probe HOME=/var/www/ooni-probe /var/www/ooni-probe/ooniprobe list           # recent results
```

Results live in `/var/www/ooni-probe/.ooniprobe/`. Uninstall removes the user,
service, timer, logo, and data.
