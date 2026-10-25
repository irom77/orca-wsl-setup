# Agent instructions

Read [ORCA-WSL-SETUP.md](ORCA-WSL-SETUP.md) before changing or troubleshooting this Orca WSL setup. It records the service commands, profile, network assumptions, recovery steps, and last verified client and CLI versions.

The Orca remote runtime runs in Ubuntu on WSL as the `orca-server.service` systemd user service. Its profile is `~/.config/orca`, its CLI is `~/.local/bin/orca-ide`, and it listens on port `6768`. The setup guide records the last verified Windows client and WSL CLI versions. When asked for the current Windows client version, inspect the Windows installation using the PowerShell command in the guide; do not substitute the WSL CLI version or assume the recorded version is current. If the Windows install cannot be inspected from the available environment, ask the user to check it in Orca's About screen or provide the installed version. Update the guide when a newer version is verified.

Keep this repository safe to publish: never add pairing URLs, device tokens, credentials, or literal machine IP addresses. Do not claim Orca Mobile is paired unless it has been verified; mobile pairing is a separate setup described in the README and the linked Orca documentation.

Keep machine-specific setup details in `ORCA-WSL-SETUP.md`. Keep `README.md` as the public overview and connection diagram.
