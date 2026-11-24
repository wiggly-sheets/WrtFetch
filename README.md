# WrtFetch
<img width="677" height="537" alt="SCR-20261106-traw" src="https://github.com/user-attachments/assets/99c7b45b-4ec9-40fd-a0a8-577f6cfebc5c" />

A lightweight, Fastfetch-inspired system information script for **OpenWrt routers**.

WrtFetch displays useful information about your router, including its hardware, CPU, memory, storage, network interfaces, Wi-Fi clients, uptime, and live WAN traffic.

Designed for portability, WrtFetch uses lightweight shell commands and aims to work with BusyBox `ash` without requiring Bash or heavyweight dependencies.

## Installation

WrtFetch is a standalone shell script. No package installation is required.

**Download directly on your router:**

```sh
wget -O /usr/bin/wrtfetch https://raw.githubusercontent.com/wiggly-sheets/WrtFetch/main/wrtfetch
chmod +x /usr/bin/wrtfetch
wrtfetch
```

Alternatively, transfer it from your computer using SCP:

```sh
scp -O wrtfetch root@192.168.1.1:/usr/bin/wrtfetch
```

Then SSH into your router and make it executable:

```sh
chmod +x /usr/bin/wrtfetch
wrtfetch
```

## Optional Setup

**Run automatically when opening an SSH session**

Edit the root user's `.profile`:

```sh
vi /root/.profile
```

Add:

```sh
case "$-" in
    *i*) wrtfetch ;;
esac
```

This runs WrtFetch when an interactive shell starts, without interfering with non-interactive SSH commands or file transfers.

Remove the added lines to disable automatic startup.

**Disable the default OpenWrt login banner (optional)**

If you prefer to see WrtFetch without the standard OpenWrt welcome banner, you can disable it without deleting or modifying `/etc/banner`.

Edit `/etc/profile`:

```sh
vi /etc/profile
```

Find this line:

```sh
[ -f /etc/banner ] && cat /etc/banner
```

Comment it out:

```sh
# [ -f /etc/banner ] && cat /etc/banner
```

Save and reconnect over SSH. The default OpenWrt banner will no longer appear.

To restore it, simply remove the `#`.

## Persistence Across Firmware Updates

The install commands above put the script in `/usr/bin`. That works, but `sysupgrade` does not keep the whole filesystem. It saves only a whitelist: `/lib/upgrade/keep.d/*` plus your `/etc/sysupgrade.conf`. `/usr/bin` is not on that list, so the script is lost on the next upgrade.

This section is optional. Use it if you want WrtFetch to survive `sysupgrade`.

**1. Store the script under `/etc`**

```sh
wget -O /etc/wrtfetch https://raw.githubusercontent.com/wiggly-sheets/WrtFetch/main/wrtfetch
chmod +x /etc/wrtfetch
```

**2. Whitelist the script and the shell hooks**

Append to `/etc/sysupgrade.conf`:

```
/etc/wrtfetch
/root/.profile
/etc/profile
```

Only list the files you actually edited. If you skipped the `.profile` and banner setup, list only `/etc/wrtfetch`.

**3. Recreate the `wrtfetch` command after each upgrade**

`/etc/uci-defaults/` survives `sysupgrade` and runs once on first boot. Use it to restore the symlink:

```sh
cat > /etc/uci-defaults/99-wrtfetch <<'EOF'
#!/bin/sh
ln -sf /etc/wrtfetch /usr/bin/wrtfetch
exit 0
EOF
chmod +x /etc/uci-defaults/99-wrtfetch
/etc/uci-defaults/99-wrtfetch
```

**4. Verify**

```sh
sysupgrade -l
```

`sysupgrade -l` prints the files that will be included in the backup. `/etc/wrtfetch` should appear in the list.

Keep "Keep configuration files" checked during the upgrade. Flashing with `sysupgrade -n` wipes the overlay and everything above.

## Compatibility

- Designed for OpenWrt and BusyBox `ash`.

- No Bash dependency.

- No additional packages required by default.

- Intended for resource-constrained routers.

- Hardware detection uses available system and device-tree information where possible.


Some information may be unavailable on certain devices. Missing values should gracefully fall back to `Unknown` or `N/A`.

## Customization and Contributions

WrtFetch is intended to be hardware-agnostic, but OpenWrt runs on a wide variety of hardware. Detection methods may differ between manufacturers, architectures, and device families.

Contributions are welcome!

If you have a device that reports information incorrectly or exposes hardware details differently, feel free to submit a pull request.

When contributing:

- Prefer generic detection methods over device-specific hardcoding.

- Keep compatibility with BusyBox `ash`.

- Avoid unnecessary dependencies and resource-intensive commands.

- Include sensible fallbacks when information is unavailable.

- Test changes on your hardware before submitting a PR.

- Document any hardware-specific quirks or assumptions.


To customize WrtFetch, edit the script directly. Hardware detection, formatting, and the `print_info()` function are good places to start.

When adding or removing displayed information, remember to update `INFO_ROWS` so the output remains properly aligned.

## License

MIT
