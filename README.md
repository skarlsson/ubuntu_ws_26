# ubuntu_ws_26

Ansible setup that turns a fresh Ubuntu 26.04 desktop into a dev workstation
(Docker, build tools, Chrome, Signal, rustup, uv, Claude Code). Replaces ubuntu_24_ws.

```
sudo apt update && sudo apt install -y ansible git
git clone https://github.com/skarlsson/ubuntu_ws_26.git && cd ubuntu_ws_26
ansible-playbook initial-ubuntu.yml -K
```

- `-K` is only needed the first time: the role adds `%sudo ALL=(ALL) NOPASSWD: ALL` to `/etc/sudoers`.
- Log out and back in afterwards for the docker group and the new PATH entries (`~/.cargo/bin`, `~/.local/bin`).
- Override with `-e`, e.g. `-e ws_disable_swap=false`. Defaults (`roles/ubuntu_ws/defaults/main.yml`):
  - `ws_user`: the login user running the playbook (gets groups + user-level tools)
  - `ws_disable_swap: true` (swapoff + remove `/swap.img`)
  - `ws_install_rustup: true`
  - `ws_install_uv: true`
  - `ws_install_claude_code: true`
  - `ws_matrix_saver: true` / `ws_matrix_idle_seconds: 180` (Matrix-rain screensaver as a user service; `matrix-saver --now` shows it immediately)
  - `ws_space_wallpapers: true`: NASA space wallpaper slideshow in `~/Pictures/space-wallpapers`, refreshed weekly by a user timer; `space-wallpapers --refresh` picks new ones now
  - `ws_hide_desktop_home: true`: no Home folder icon on the GNOME desktop (`org.gnome.shell.extensions.ding show-home false`)
  - `ws_claude_config_repo: ""` / `ws_claude_config_dir` (`~/.claude-config`) / `ws_claude_config_links: [CLAUDE.md, agents]`: set the repo to a git URL holding `CLAUDE.md`, `agents/` and `settings.base.json`; the role clones it to `~/.claude-config`, symlinks `CLAUDE.md` and `agents` into `~/.claude` and merges `settings.base.json` into `settings.json`.
    It pulls only when the working tree is clean, so local edits are never lost. Empty (the default) skips it.
  - `ws_packages`: the apt package list
  - `ws_agent_packages`: CLI tools coding agents lean on (gh, shellcheck, yq, fzf, delta, nodejs/npm, ...)
  - `ws_install_dev_packages: true` / `ws_dev_packages`: toolchains the projects install by hand (ARM gcc, multilib, LLVM, QEMU, CAN, GTK/WebKit for Tauri, GStreamer, analysis tools)
  - `ws_i386: true` / `ws_i386_packages`: i386 dpkg architecture + 32-bit libc/libstdc++/zlib (i686 targets)
  - `ws_nofile_limit: 65536`: soft/hard open-files limit for all users (`/etc/security/limits.d/90-ws-nofile.conf`)
  - `ws_modules: [vcan]`: kernel modules loaded now and at boot (`/etc/modules-load.d/ws.conf`)
  - `ws_rust_targets`: extra rustup targets (i686, aarch64, thumbv7em/thumbv7m bare metal)
  - `ws_rust_extra_toolchains: [nightly]`: extra toolchains, each with `rust-src`
  - `ws_cargo_tools`: `cargo install`ed tools (binary name == crate name)
  - `ws_bazelisk_version`: bazelisk release installed as `~/.local/bin/bazelisk` with a `bazel` symlink
  - `ws_install_awscli: true`: AWS CLI v2, system-wide under `/usr/local`
- `roles/ubuntu_ws` is the reusable part and is also consumed by another repo as a git submodule.
  Run the role as the user being set up (no play-level `become`); it escalates only for system tasks.

## Not covered on purpose

These stay in the projects' own install scripts:

- ESP-IDF and the Espressif udev rules
- Pico SDK / picotool
- Go
- QNX SDP and its nightly tier-3 target tricks
- anything from a corporate GitLab (registries, tokens, toos, ivshmem-server, xcp-gui)
- Flutter checkouts
- per-project Python venvs
- project network setup (vcan interfaces, tap devices, QEMU sysctl/setcap)
