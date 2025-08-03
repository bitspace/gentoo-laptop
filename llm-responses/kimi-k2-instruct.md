# System76 Kudu Custom LLM-Friendly Developer Gaming Laptop Build

## Top 5 distributions (ranked by how well they satisfy *all* constraints, not just “rolling release”)

1. NixOS unstable  
   • Purely declarative (`configuration.nix` + flakes) – easily scripted, diffable, and LLM-parsable.  
   • Rolling release, every package rebuilds daily.  
   • Binary cache covers >95 % of what you need; local rebuild only when you explicitly override.  
   • Full systemd, Wayland/X11 both trivially available.  
   • Steam, Proton-GE, CUDA, ROCm all packaged; switch between versions with a one-line override.  
   • Entire system state is a text file – perfect for agent orchestration.

2. Arch Linux (or Arch-based EndeavourOS if you want the installer)  
   • Native pacman + AUR gives you the largest rolling software pool.  
   • No “compilation by default”; binary packages are the norm.  
   • systemd is the standard.  
   • Configuration is plain text (`/etc`, `~/.config`) – still LLM-friendly.  
   • Downside: imperative package management; reproducibility is only as good as your discipline.

3. openSUSE Tumbleweed  
   • rpm-ostree-style snapshots (`snapper + btrfs`) give you automatic roll-back if an update breaks anything.  
   • Uses `dnf` (libzypp backend), so binary packages only.  
   • ` transactional-update` lets you treat even `/etc` atomically.  
   • Slightly higher friction for non-packaged software (Flatpak/AUR-like OBS repos exist but are smaller).

4. Void Linux (glibc)  
   • Completely systemd-free upstream, but you can replace runit with systemd from their repo.  
   • Rolling, binary by default, xbps-src when you need source.  
   • Smaller community, so you’ll package more things yourself.

5. Gentoo with binpkgs (hybrid approach)  
   • Use the official Gentoo binhost (amd64, ~amd64) for everything except the few packages you really want optimized.  
   • `EMERGE_DEFAULT_OPTS="--usepkg=y --getbinpkg=y"` keeps compile times near zero.  
   • Still gives you USE-flag flexibility *only* for the packages you choose to build.  
   • Draw-back: you still have to maintain a portage tree and occasional blockers; LLM agents struggle with the interactive parts of `emerge`.

## Window-manager / compositor shortlist (all Wayland-first, text-configurable, lightweight)

1. Sway – drop-in i3 replacement, pure Wayland, config is `~/.config/sway/config` (i3 syntax).  
2. Hyprland – dynamic tiling, animations, plugin system; config is `hyprland.conf`.  
3. river – Zig-based, tags instead of workspaces; config in Lua.  
4. Qtile Wayland backend – entire WM written/configured in Python.  
5. Wayfire – floating/compositor hybrid, config is INI; good X11 fallback via XWayland.

## Installation roadmap (agent-orchestrated, entirely scriptable)

Phase 0 – prepare host for agent  
0.1 Boot latest Arch ISO (has `openssh` and `cloud-init`)  
0.2 Start `sshd`, set a temporary password, let the LLM agent SSH in.

Phase 1 – disk & base system  
1.1 Scripted `gdisk` → create identical layout as requested (ESP, swap, XFS root).  
1.2 `mkfs.xfs`, `mkswap`, `mount`, `swapon`.  
1.3 Install `systemd-bootctl install` → `loader.conf` and `arch.conf` entries.  
1.4 Pacstrap minimal set: `base linux linux-firmware sof-firmware neovim git openssh`.  
1.5 Chroot in, set root password, create user with sudo.

Phase 2 – pick the final distribution  
Agent evaluates constraints again (compile-time, declarative, binary cache hit-rate).  
If the agent chooses NixOS:  
2a. `nixos-generate-config --root /mnt` → edit `/mnt/etc/nixos/configuration.nix` directly from the agent.  
2b. Enable flakes, add `nixpkgs.config.allowUnfree = true;`, enable `steam`, `programs.wayland.wm.sway.enable = true`, GPU drivers (`hardware.nvidia.open = true` + `hardware.amdgpu.initrd = true`).  
2c. `nixos-install --root /mnt`.

If the agent sticks with Arch:  
2d. Continue normal Arch install, then configure declarative dotfiles via `chezmoi` or `yadm` pulled from a git repo.

Phase 3 – graphical stack  
3.1 Install GPU drivers:  
   • AMD: `mesa`, `libva-mesa-driver`, `vulkan-radeon`.  
   • Nvidia: `nvidia-dkms`, `egl-wayland`, `libva-nvidia-driver`.  
3.2 Install chosen compositor (example: `sway`, `foot`, `bemenu`, `waybar`).  
3.3 Systemd user services for `pipewire`, `wireplumber`, `xdg-desktop-portal-wlr`.

Phase 4 – development & AI stack  
4.1 `nix-shell -p python311 rustup nodejs_20 gcc clang glibc` (or `pacman -S …`).  
4.2 Install `pyenv`, `rustup`, `nvm`, `ghcup` via user-level scripts.  
4.3 Pull `ollama` or `ollama-nvidia` container for local LLM inference.  
4.4 Configure `docker`, `podman`, `kind`, `kubectl`, `gcloud`, `aws-cli-v2` via declarative module (NixOS) or `yay` (Arch).  
4.5 Steam + Proton-GE: `steam`, `proton-ge-custom-bin` (AUR) or `programs.steam.enable = true;` in NixOS.

Phase 5 – music production  
5.1 Install `pipewire-jack`, `carla`, `ardour`, `yabridge`, `lmms`.  
5.2 Add user to `realtime` group, install `rtirq` (only if low-latency kernel needed).  
5.3 All configs live in `~/.config` – versioned in the same dotfiles repo.

Phase 6 – LLM agent hooks  
6.1 `systemctl --user enable agent-resume.service` that pulls latest dotfiles on boot.  
6.2 Provide the agent a JSON schema of every editable config file (generated from `nixos-option`, `chezmoi data`, or `sway --get-socketpath`).  
6.3 Add `inotifywait` triggers so the agent can auto-reload changed configs.

Phase 7 – snapshot & rollback  
NixOS: every `nixos-rebuild switch` is automatically in the boot menu.  
Arch: `snapper` on `/` + `grub-btrfs` for instant roll-back.

Phase 8 – documentation artifact  
After first successful boot, agent outputs a single markdown file containing:  
• exact disk layout, LUKS keyfile location, kernel cmdline,  
• flake URL and lockfile hash,  
• systemd service list with `systemctl cat` output,  
• declarative list of packages (`nix-store -q --requisites /run/current-system`).  
Result: an LLM can reproduce the entire system with two commands (`nix flake clone` + `nixos-rebuild switch --flake .#hostname`).
