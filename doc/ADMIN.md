No web UI. Runs as a systemd timer
(`systemctl list-timers 'ooni-probe*'`); last run results and logs:
`yunohost service log ooni-probe`. Tune the URL cap and suite via app
settings.
