# ooni-probe — packaging design

OONI Probe measures internet censorship. The package is not a web app: it drops a single static binary and a systemd timer that runs it every N hours, and there is no UI, no port, and no login anywhere. The headless timer shape drives everything below.

Anything not mentioned here is standard YunoHost boilerplate identical in most packages.

## Lifecycle

- **Install:** download the pinned [per-arch binary](../manifest.toml#L43-L50), write a pre-seed [consent config](../scripts/install#L14-L18), create the state dirs under `$install_dir/.ooniprobe`, render the [oneshot service and timer](../scripts/install#L25-L31), enable the timer, and start it. The first measurement run begins within ten minutes of boot or immediately via the timer start.
- **Upgrade:** re-run `ynh_setup_source` ([comment says why](../scripts/upgrade#L5-L6): a rolled-back install can deprovision sources), delete then re-render config and both units, restart the timer. The state dir survives because the helper extracts in place, not with `--full_replace`.
- **Backup/restore:** archives [config, db, and both unit files](../scripts/backup#L5-L8). Restore [re-enables and restarts the timer](../scripts/restore#L14-L21).

## Decisions

- **Source:** [per-arch release binaries, sha256-pinned](../manifest.toml#L43-L50), `extract=false` + `rename=ooniprobe` so `ynh_setup_source` just places the file.
- **Runtime:** static Go binary, no interpreter, no compile step, and no `[resources.apt]` at all. Zero system packages.
- **Services:** one oneshot service plus a timer, no long-running daemon. The timer fires [ten minutes after boot and every `run_interval_hours`](../conf/systemd.timer#L5-L8), with 30 min random jitter and `Persistent=true` so a reboot that missed a run catches up.
- **Storage:** the app keeps state in `$install_dir/.ooniprobe` (config, db, measurements) because [HOME is set to the install dir](../conf/systemd.service#L10) and upstream writes to `$HOME/.ooniprobe`. There is no `data_dir` resource, which is why backup and restore name the paths by hand.
- **Auth:** none. The manifest [declares ldap/sso not_relevant](../manifest.toml#L22-L23) and defines no permissions or web path; nothing can reach the box through this app.
- **Ports/firewall:** none. The binary only makes outbound connections during a run.
- **Scheduler:** the timer is the product. The [install question `run_interval_hours`](../manifest.toml#L28-L31) is rendered into the timer unit, and [`url_limit`](../manifest.toml#L33-L36) into the [config template](../conf/ooniprobe-config.template#L8-L9).
- **Migration:** none needed; a version bump is a binary swap.
- **Sandboxing:** the strictest of our packages, and it can be, because nothing here needs privileges: [full Protect flags, DevicePolicy closed, capability denylist, address-family allowlist, MemoryMax 400M, Nice=10](../conf/systemd.service#L13-L29). Note `ProtectHome=yes` is safe here only because HOME points at `/var/www`, not `/home`.
- **Backup:** config.json and the sqlite db are archived; the `msmts/` measurements dir is not. No comment in the script explains the omission.
- **Admin-surface:** no config panel. The only knobs are the two install questions; changing them means editing app settings and re-running upgrade so the templates re-render.

## Don't break

- [Delete config.json and the timer before `ynh_config_add`](../scripts/upgrade#L9-L15) on upgrade and restore. The helper checksum-skips unchanged templates; without the delete, an edited `url_limit` would silently not apply.
- [Re-render the service on upgrade, not just the timer](../scripts/upgrade#L16-L17). The in-file comment says systemd refuses a timer whose unit is missing.
- [Restore both unit files](../scripts/restore#L12-L18). A restore onto a fresh machine has no unit files, and without them the timer never runs again.
- The [`|| true` on the portal-logo permission update](../scripts/install#L36-L40). The manifest defines no permissions, so the update is best-effort decoration; without the guard, installs die on a cosmetic copy.
