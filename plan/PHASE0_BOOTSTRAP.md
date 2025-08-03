# Phase 0 — Bootstrap Plan

**Goal:** Prepare the target machine (currently bare Gentoo) for a fresh NixOS installation and establish a minimal environment where the OpenAI Codex CLI and orchestration scripts can run.

**Confirmation Requirements:**
- Any destructive disk or bootloader operations require `@confirm:critical` in the task script.
- Every `sudo` invocation must be preceded by an explicit `# @confirm:` comment.

## Tasks

### 0.1 Download & Verify NixOS Installer ISO
- Download and validate the NixOS installer image:
  ```bash
  # @confirm:reason="retrieve NixOS installer image",scope="user"
  ISO_URL="https://releases.nixos.org/nixos/latest-nixos-minimal-x86_64-linux.iso"
  SIG_URL="${ISO_URL}.sig"
  curl -LO "$ISO_URL" "$SIG_URL"
  gpg --fetch-keys https://nixos.org/keys/nix-signing-key.pub
  gpg --verify "$(basename $SIG_URL)" "$(basename $ISO_URL)"
  sha256sum --check <(curl -s "${ISO_URL}.sha256")
  ```

### 0.2 Create Bootable USB Media
- Burn the ISO to USB (adjust `/dev/sdX` accordingly):
  ```bash
  # @confirm:reason="create bootable USB media",scope="system"
  sudo dd if=$(basename $ISO_URL) of=/dev/sdX bs=4M status=progress && sync
  ```

### 0.3 Boot Installer & Partition Disks
- Boot the machine from USB and partition the primary disk:
  ```bash
  # @confirm:critical
  sudo parted /dev/sdY --script \
    mklabel gpt \
    mkpart primary 1MiB 512MiB \
    set 1 boot on \
    mkpart primary 512MiB 100%
  ```

### 0.4 Format & Mount Filesystems
- Format EFI and root partitions, then mount for install:
  ```bash
  sudo mkfs.fat -F32 /dev/sdY1
  sudo mkfs.ext4 /dev/sdY2
  sudo mount /dev/sdY2 /mnt
  sudo mkdir -p /mnt/boot
  sudo mount /dev/sdY1 /mnt/boot
  ```

### 0.5 Generate & Review NixOS Configuration
- Generate the default NixOS config and add minimal settings:
  ```bash
  sudo nixos-generate-config --root /mnt
  # Edit /mnt/etc/nixos/configuration.nix to include:
  #   networking.hostName = "<your-host>";
  #   time.timeZone = "UTC";
  #   boot.loader.systemd-boot.enable = true;
  #   fileSystems."/".device = "/dev/sdY2";
  #   fileSystems."/boot".device = "/dev/sdY1";
  ```

### 0.6 Install NixOS & Initial Reboot
- Install and reboot into the new NixOS system:
  ```bash
  sudo nixos-install --no-root-password
  sudo reboot
  ```

### 0.7 Workspace Initialization (post-boot)
- After rebooting into NixOS, create the project workspace under `$HOME/llm-laptop/`:
  ```bash
  mkdir -p $HOME/llm-laptop/{plan/DECISION_RECORDS,infra,scripts,reports,logs,llm-responses,constraints,preferences,secrets}
  touch $HOME/llm-laptop/plan/CONSENSUS.md
  touch $HOME/llm-laptop/FACTS.json $HOME/llm-laptop/STATE.json
  chmod 700 $HOME/llm-laptop/secrets
  ```

### 0.8 Capture System Facts
- Collect hardware & OS facts in JSON for future phases:
  ```bash
  # @confirm:reason="capture hardware facts",scope="system"
  set -Eeuo pipefail
  {
    echo '{'
    echo '  "lspci": '$(lspci -nnk | jq -Rs .)',
    echo '  "lsusb": '$(lsusb | jq -Rs .)',
    echo '  "uname": '"$(uname -a)"',
    echo '  "os_release": '$(jq -Rs . </etc/os-release)',
    echo '  "df": '$(df -h | jq -Rs .)',
    echo '  "free": '$(free -h | jq -Rs .)',
    echo '  "nproc": '$(nproc)',
    echo '  "lscpu": '$(lscpu | jq -Rs .)',
    echo '  "lsblk": '$(lsblk -o NAME,SIZE,TYPE,MOUNTPOINT | jq -Rs .)',
    echo '  "vulkaninfo": '$(vulkaninfo || true | jq -Rs .)',
    echo '  "nvidia_smi": '$(nvidia-smi || true | jq -Rs .)',
    echo '  "glxinfo": '$(glxinfo -B | jq -Rs . || true)'
    echo '}'
  } >$HOME/llm-laptop/FACTS.json
  ```

### 0.9 Install Prerequisite Tooling
- Install core build & scripting tools (git, curl, jq, yq, ripgrep, fd, make, gcc, clang, pkg-config) and Python:
  ```bash
  # @confirm:reason="install prerequisite tooling",scope="system"
  nix-env -iA nixpkgs.git nixpkgs.curl nixpkgs.jq nixpkgs.yq nixpkgs.ripgrep \
    nixpkgs.fd nixpkgs.make nixpkgs.gcc nixpkgs.clang nixpkgs.pkg-config nixpkgs.python3
  ```

### 0.10 Install OpenAI Codex CLI
- Use pipx to install the Codex CLI and verify authentication:
  ```bash
  # @confirm:reason="install Codex CLI",scope="user"
  pipx install openai-codex-cli || pipx install openai
  openai auth status
  ```

### 0.11 Network, NTP & Timezone
- Validate connectivity and enable NTP:
  ```bash
  ping -c1 8.8.8.8
  sudo timedatectl set-ntp true
  timedatectl status
  ```

### 0.12 Checkpoint Bootstrap State
- Write `STATE.json` to signal completion of Phase 0:
  ```bash
  jq -n '{phase: "BOOTSTRAPPED", timestamp: now}' >$HOME/llm-laptop/STATE.json
  ```

### 0.13 Logging
- Append all Phase 0 commands & outputs to a timestamped log:
  ```bash
  exec &> >(tee -a $HOME/llm-laptop/logs/commands-$(date +"%Y%m%d-%H%M%S").log)
  ```

---

*Next:* Phase 1 — Ingest & Normalize (requires Codex CLI installation complete)
