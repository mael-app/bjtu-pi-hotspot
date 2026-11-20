# BJTU Pi hotspot

> **中国同学？中文说明在这里 👉 [简体中文](README.zh-CN.md)**

A Raspberry Pi 4 acts as a gateway: it logs in to the university (BJTU) captive portal by itself, shares Internet over a LAN, and updates itself at night.
Everything is deployed by Ansible from your laptop, onto a fresh Pi.

```
University --> [eth0  Raspberry Pi  eth1] --> TP-Link --> devices
  (portal)       auto login      192.168.50.1/24   hotspot
```

## Laptop prerequisites

1. `ansible-core` installed.
2. Collections: `ansible-galaxy collection install community.general ansible.posix`

## Set up a fresh Pi

1. Flash **Raspberry Pi OS Lite 64-bit** with Raspberry Pi Imager, **with no customisation** (no user, no wifi).
2. `cp authorized_keys.example authorized_keys`, then add your public SSH key(s). The file is gitignored.
3. Put the SD card back in the laptop, then:
   ```
   ansible-playbook bootstrap.yml -e boot=/path/to/bootfs
   ```
   This writes `user-data` (cloud-init) to the boot partition: root SSH key (from the local `authorized_keys` file), SSH enabled, no user, and the hotspot LAN on eth1 (192.168.50.1, DHCP for clients).
4. Plug eth0 into the university network and eth1 (USB adapter) into the TP-Link, boot the Pi, and connect the laptop to the TP-Link wifi. The Pi is at `192.168.50.1`. There is no Internet yet (portal not passed): turn off any VPN on the laptop.
5. `cp .env.example .env`, then fill in `BJTU_USERNAME` and `BJTU_PASSWORD`.
   `.env` is gitignored and holds the portal credentials. It is copied to the Pi as `/etc/bjtu-login.env` (mode 0600).
6. `inventory/hosts.yml` already targets `192.168.50.1`: nothing to change.
7. First run (default SSH port 22):
   ```
   ansible-playbook site.yml -e ansible_port=22
   ```
8. Later runs (port 22422):
   ```
   ansible-playbook site.yml
   ```

Need to open more ports? Use `firewall_extra_tcp_ports` / `firewall_extra_udp_ports` in `group_vars/all.yml`.

## What gets deployed

| Role | What it does |
|---|---|
| `bjtu_login` | Installs `/usr/local/bin/bjtu-login`, `/etc/bjtu-login.env` (0600), a systemd service + timer and a NetworkManager dispatcher (re-login when eth0 comes up). Also does the first portal login if there is no Internet. |
| `base` | Timezone and base packages (`curl`, `nftables`, `fail2ban`, `network-manager`, ...) |
| `network` | Manages only eth1: NetworkManager connection `lan_connection` (`hotspot-lan`), `lan_address` (192.168.50.1/24), shared mode (NAT + DHCP/DNS). eth0 is left on NetworkManager's default DHCP. |
| `firewall` | nftables inbound rules (table `rpi_guard`: eth1 trusted, SSH + extra ports open, rest dropped) and fail2ban for SSH |
| `autoupdate` | Nightly unattended-upgrades, reboot only if required |
| `ssh` | Port 22422, key-only. Root `authorized_keys` is managed exclusively from the local `authorized_keys` file. Runs last. |

## Nightly schedule

| Time | Action |
|---|---|
| 03:00 | `apt update` |
| 04:00 | `apt upgrade` (unattended-upgrades) |
| 05:00 | Reboot, only if an update requires it |

Change the times in `group_vars/all.yml`.

## Troubleshooting

- Login logs: `journalctl -u bjtu-login`
- Next timers: `systemctl list-timers`
- Force a login: `systemctl start bjtu-login`
- Preview before a run: `ansible-playbook site.yml --check --diff`

## License

[MIT](LICENSE). Contributions welcome: see [CONTRIBUTING.md](CONTRIBUTING.md).
