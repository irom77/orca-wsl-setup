# Agent instructions

Read [ORCA-WSL-SETUP.md](ORCA-WSL-SETUP.md) before changing or troubleshooting this Orca WSL setup. It records the service commands, profile, network assumptions, recovery steps, and last verified client and CLI versions.

The Orca remote runtime runs in Ubuntu on WSL as the `orca-server.service` systemd user service. Its profile is `~/.config/orca`, its CLI is `~/.local/bin/orca-ide`, and it listens on port `6768`.

Keep the WSL Orca runtime version in sync with the installed Windows Orca client. Before upgrading or downgrading Orca on WSL, inspect the Windows installation using the PowerShell command in `ORCA-WSL-SETUP.md`, then install that same Orca release on WSL. Do not substitute the WSL CLI version for the Windows client version or assume the guide's recorded version is current. If the Windows install cannot be inspected from the available environment, ask the user to check Orca's About screen or provide the installed version before changing WSL. After an update, verify both versions and update the setup guide.

Keep this repository safe to publish: never add pairing URLs, device tokens, credentials, or literal machine IP addresses. Do not claim Orca Mobile is paired unless it has been verified; mobile pairing is a separate setup described in the README and the linked Orca documentation.

Keep machine-specific setup details in `ORCA-WSL-SETUP.md`. Keep `README.md` as the public overview and connection diagram.
