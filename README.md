<div align="center">

# yoga11e-cyberdeck

**A decade-old Lenovo Yoga 11e, stripped down to a pure-terminal dashboard.**

No X11. No Wayland. No desktop environment. Just `tty1`, `tmux`, and Python.

[![Debian](https://img.shields.io/badge/Debian-13_trixie-A81D33?style=flat-square&logo=debian&logoColor=white)](https://www.debian.org/)
[![Python](https://img.shields.io/badge/Python-3-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![tmux](https://img.shields.io/badge/tmux-persistent_session-1BB91F?style=flat-square&logo=tmux&logoColor=white)](https://github.com/tmux/tmux)
[![Tailscale](https://img.shields.io/badge/Tailscale-mesh_networking-4B5BFF?style=flat-square&logo=tailscale&logoColor=white)](https://tailscale.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-000000?style=flat-square)](LICENSE)

🇬🇧 English&nbsp;&nbsp;|&nbsp;&nbsp;[🇵🇱 Polski](README.pl.md)

</div>

---

## What this is

`yoga11e-cyberdeck` turns an old Lenovo Yoga 11e (Bay Trail, Celeron N2940, 4 GB RAM) into a
dedicated, always-on terminal console — a live system dashboard that boots directly into a
full-screen `tty1`, no login prompt, no graphical stack.

It's built around a simple idea: the hardware is too weak and too old to run a modern desktop
comfortably, but it's more than capable of being a beautiful, information-dense terminal.

## Screenshot

*(insert your own screenshot here — `dashboard.py` running on tty1)*

## Stack

- **Debian 13 (trixie)** — pure console, no display server at all
- **Python 3 + [`rich`](https://github.com/Textualize/rich)** — the dashboard itself
- **tmux** — persistent session, survives SSH disconnects
- **systemd** — autologin scoped to `tty1` only (`tty2`–`tty6` stay clean logins)
- **Tailscale** — mesh access to the rest of the fleet, status shown live on the dashboard

## Features

- Live system info panel (OS, kernel, uptime, packages, CPU, memory, disk, battery)
- Debian ASCII logo, colored to match hardware/software state
- Per-core CPU load, memory/swap/battery/disk bars — all threshold-colored (green → yellow → red)
- Network throughput (up/down)
- Live reachability check for other machines on the Tailscale mesh
- Top processes table
- Shortcuts / quick-commands / recent-commands reference panels
- Autologin scoped to `tty1` — other virtual terminals (`Alt`+`F2`..`F6`) stay untouched for normal work

## Hardware

| | |
|---|---|
| Model | Lenovo Yoga 11e (20DU) |
| CPU | Intel Celeron N2940 (Bay Trail, 4 cores) @ 1.83 GHz |
| RAM | 4 GB |
| Storage | eMMC |
| WiFi | Intel Wireless 7260 (`iwlwifi`) |

Bay Trail-era hardware has a reputation for being awkward under Linux. In practice, on Debian 13
the only real friction points were:
- `iw` lives in `/usr/sbin`, which isn't on a regular user's `$PATH` — call it with the full path
- WiFi firmware for the 7260 is already bundled via `non-free-firmware` in modern Debian installers
- keep Ethernet and WiFi both under NetworkManager (`managed=true`), or you'll get inconsistent DNS
  when switching between them

## Install

```bash
sudo apt install -y python3-rich python3-psutil tmux figlet cmatrix curl

mkdir -p ~/.config/dashboard ~/.config/tmux
cp dashboard.py       ~/.config/dashboard/dashboard.py
cp tmux.conf          ~/.config/tmux/tmux.conf
cp boot.sh            ~/.config/tmux/boot.sh
cp session.sh         ~/.config/tmux/session.sh
chmod +x ~/.config/dashboard/dashboard.py ~/.config/tmux/*.sh
echo "source-file ~/.config/tmux/tmux.conf" >> ~/.tmux.conf

cat profile-snippet.sh >> ~/.bash_profile

sudo mkdir -p /etc/systemd/system/getty@tty1.service.d
sudo cp getty-override.conf /etc/systemd/system/getty@tty1.service.d/override.conf
# edit the username inside getty-override.conf first
sudo systemctl daemon-reload
sudo systemctl restart getty@tty1
```

Edit `dashboard.py` to point `THINKCENTRE_HOST` / `ORANGEPI_HOST` (or add your own) at whatever
hostnames show up in `tailscale status` for your own mesh.

## Architecture

```
        yoga11e (tty1)
             |
        tmux session
             |
        dashboard.py
             |
      Tailscale mesh
     /       |        \
 server   orangepipc2  ...
(ThinkCentre)
```

## License

MIT — see [LICENSE](LICENSE).
