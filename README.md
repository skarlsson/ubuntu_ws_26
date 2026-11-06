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
  - `ws_packages`: the apt package list
- `roles/ubuntu_ws` is the reusable part and is also consumed by another repo as a git submodule.
  Run the role as the user being set up (no play-level `become`); it escalates only for system tasks.
