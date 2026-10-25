# Orca remote runtime on WSL

Notes for the Orca Linux runtime used by the Orca client on Windows.

## Installation details

- Orca site: <https://www.onorca.dev/>
- WSL distro: Ubuntu
- Orca CLI: `~/.local/bin/orca-ide`
- Orca profile: `~/.config/orca`
- Runtime: Electron
- Windows Orca client version at last check: `1.4.240`
- WSL Orca CLI version at last check: `1.4.227`
- Tailscale address: private; use the current address shown by `tailscale status --self`.
- Runtime port: `6768`

The Windows client is paired to this runtime already. The pairing URL contains a device token and should be treated like a password. Do not put the URL, token, or other pairing secrets in this document, shell history, or shared logs. The service uses `--no-pairing`, so a normal restart does not create a new pairing offer.

## Managed service

The runtime is run as a systemd **user** service at:

```text
~/.config/systemd/user/orca-server.service
```

The unit starts the Electron runtime in a headless display and a private D-Bus session:

```text
/usr/bin/dbus-run-session -- /usr/bin/xvfb-run -a \
  %h/.local/bin/orca-ide serve \
  --port 6768 --no-pairing --json
```

Useful controls (run as user `irom`, without `sudo`):

```bash
systemctl --user status orca-server.service --no-pager
systemctl --user restart orca-server.service
systemctl --user stop orca-server.service
systemctl --user start orca-server.service
journalctl --user -u orca-server.service -n 50 --no-pager
journalctl --user -u orca-server.service -f
```

The service is enabled and the user has lingering enabled. On this machine it starts when the WSL distro starts. After a full Windows reboot, launch Ubuntu WSL at least once if it did not start automatically.

## Health checks

Run these in Ubuntu WSL:

```bash
systemctl --user is-active orca-server.service
ss -ltnp '( sport = :6768 )'
curl -sS --max-time 3 http://localhost:6768/ -o /dev/null \
  -w 'HTTP status: %{http_code}\n'
tailscale status --self
```

Expected signs of a healthy runtime:

- `systemctl --user is-active` prints `active`.
- `ss` shows Orca listening on port `6768` on all interfaces.
- The local HTTP probe returns `200`.
- Tailscale shows this Ubuntu node at its current Tailscale address.

The service's ready event may advertise localhost even while the socket is bound on all interfaces. If the client still cannot connect while the local probe works, check Tailscale connectivity and whether the Windows client is using its saved pairing endpoint.

## Troubleshooting: “Remote runtime disconnected”

Check in this order:

1. **Is the WSL distro running?** Open Ubuntu from Windows. WSL does not always start the distro automatically after a full Windows reboot.
2. **Is the service active?** Run `systemctl --user status orca-server.service --no-pager -l`.
3. **Is port 6768 listening?** Run `ss -ltnp '( sport = :6768 )'`.
4. **Does Orca answer locally?** Run the `curl` health check above.
5. **Is Tailscale connected?** Check `tailscale status --self`; from Ubuntu, `tailscale ping zenbook` has reached the Windows peer on this machine.
6. **Inspect recent logs:** `journalctl --user -u orca-server.service -n 50 --no-pager`.

### If `systemctl --user` says “Failed to connect to bus”

This happened on Oct 24, 2026. The user manager `user@1000.service` had been killed, leaving the user D-Bus socket behind but refusing connections. The Orca service unit itself was intact. From the Ubuntu shell, restore the user manager, then start Orca:

```bash
sudo systemctl reset-failed user@1000.service
sudo systemctl start user@1000.service

systemctl --user enable --now orca-server.service
systemctl --user status orca-server.service --no-pager -l
```

The `sudo` commands prompt for the Ubuntu password. Then allow a few seconds for the Electron runtime to initialize before checking the port:

```bash
ss -ltnp '( sport = :6768 )'
curl -sS --max-time 3 http://localhost:6768/ -o /dev/null \
  -w 'HTTP status: %{http_code}\n'
```

If starting the user manager fails again, inspect `systemctl status user@1000.service --no-pager -l` and `journalctl -u user@1000.service -n 50 --no-pager`.

### If the service is active but port 6768 is not listening

An `active (running)` status captured immediately after starting can precede Orca's listener initialization. Wait a few seconds, then recheck the service, listener, and log:

```bash
sleep 3
systemctl --user status orca-server.service --no-pager -l
ss -ltnp '( sport = :6768 )'
journalctl --user -u orca-server.service -n 50 --no-pager
```

Look for the JSON `orca_server_ready` event. If it does not appear, check for errors in the service log and restart the service once.

### Tailscale connmark warning

The WSL Tailscale status once reported that iptables did not support `CONNMARK` options (`--nfmask` / `--ctmask`). At that time the node still had its expected Tailscale address, and later a ping to the Windows peer succeeded. Treat this warning as a separate networking issue if Orca listens locally but Windows cannot reach it; confirm current status before changing firewall or kernel settings.

## Manual runtime commands

The systemd service is preferred for normal operation. For a foreground pairing offer or temporary diagnosis, the previously used command was:

```bash
ORCA_SERVE_RUNTIME=electron ~/.local/bin/orca-ide serve \
  --port 6768 --pairing-address "$(tailscale ip -4)" --json
```

This can print a pairing URL with a device token. Handle its output as a secret. Avoid starting a manual server while the service is running, since both need port 6768. Do not disable `--no-pairing` on the managed service just to restart it; use the manual command only when a new pairing offer is actually needed.

## Last recovery record

On Oct 25, 2026, the Windows client showed “Remote runtime disconnected.” The Orca service had stopped listening after the systemd user manager was killed on Oct 24. Starting `user@1000.service` restored the user D-Bus; enabling and starting `orca-server.service` brought Orca back. Verification succeeded with an active service, a listener on all interfaces at port `6768`, HTTP `200` on localhost, and a successful Tailscale ping to the Windows peer. The Windows client then reconnected without pairing again.

Update this record when the service setup, address, Orca version, or recovery steps change. Never add the pairing URL or token.
