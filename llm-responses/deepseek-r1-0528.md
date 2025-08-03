# System76 Kudu Custom LLM-Friendly Developer Gaming Laptop Build

### Top 5 Distribution Recommendations
1. **NixOS (Declarative)**  
   - **Why**: Ideal for scriptability/LLM parsing due to declarative configuration. Atomic updates ensure stability. Excellent for development environments and reproducible AI/ML workflows. Hybrid graphics support via Nix expressions.
   - **Package Manager**: Nix (with flakes for versioning)
   - **Rolling Option**: `nixos-unstable` channel
   - **GPU Handling**: Pre-configured NVIDIA PRIME offloading

2. **Arch Linux (Binary)**  
   - **Why**: Bleeding-edge packages with AUR access. Pacman's simplicity aids automation. Artix (systemd variant) possible if avoiding Arch's installer.
   - **Package Manager**: Pacman + AUR helpers (e.g., `paru`)
   - **GPU Handling**: `optimus-manager` or `nvidia-prime`

3. **Gentoo (Hybrid Binary/Source)**  
   - **Why**: Balance Portage flexibility with reduced compile times via `binhost` setup. Use binaries for large packages (Chromium, LibreOffice, LLVM).
   - **Config**: Set `FEATURES="getbinpkg"` in `make.conf`
   - **GPU Handling**: `envycontrol` for GPU switching

4. **openSUSE Tumbleweed (Binary)**  
   - **Why**: Robust rolling release (`zypper`). YaST configurations are scriptable via XML. OBS provides custom packages.
   - **Package Manager**: Zypper + OBS
   - **GPU Handling**: `suse-prime` for NVIDIA

5. **Fedora Rawhide (Binary)**  
   - **Why**: Bleeding-edge with reliable SELinux integration. DNF configurations are YAML-based for LLM parsing.
   - **Package Manager**: DNF
   - **GPU Handling**: `akmod-nvidia` + `xorg-x11-drv-nvidia-power`

### Window Manager Recommendations (Wayland-Compatible)
1. **Hyprland**  
   - **Why**: Dynamic tiling with declarative config (hyprland.conf). Explicit NVIDIA support. Low-latency rendering for gaming.
   - **Automation**: Config files support variable substitution
   - **Best for**: Gaming/development balance

2. **Sway**  
   - **Why**: i3-compatible with strict Wayland support. Config uses plaintext. Battle-tested for stability.
   - **Automation**: `swaymsg` for runtime control
   - **Best for**: Development focus

3. **River**  
   - **Why**: Simple Lua-configurable tiling. Explicit NVIDIA support. Low resource footprint.
   - **Automation**: Lua scripts for dynamic layouts
   - **Best for**: GPU-heavy workflows

4. **Qtile (Wayland Preview)**  
   - **Why**: Python-configured. Extensible via plugins. Experimental Wayland support.
   - **Automation**: Full API access for scripting
   - **Best for**: Python-centric development

5. **Labwc**  
   - **Why**: Openbox-compatible with XML configs. Lightweight but XDG-compliant.
   - **Automation**: `obconf`-style declarative theming
   - **Best for**: Music production low overhead

### Installation Plan
**Phase 1: Base System (NixOS Example)**
1. Prepare installation media:  
   ```bash
   nix-shell -p nixos-install-tools
   parted /dev/nvme0n1 mklabel gpt
   parted /dev/nvme0n1 mkpart ESP fat32 1MiB 1GiB
   parted /dev/nvme0n1 mkpart swap linux-swap 1GiB 17GiB
   parted /dev/nvme0n1 mkpart root xfs 17GiB 100%
   mkfs.fat -F32 /dev/nvme0n1p1
   mkswap /dev/nvme0n1p2
   mkfs.xfs /dev/nvme0n1p3
   ```

2. Configure hardware:  
   ```nix
   # /etc/nixos/hardware.nix
   { config, ... }: {
     boot.initrd.kernelModules = ["nvidia"];
     hardware.nvidia = {
       modesetting.enable = true;
       prime = {
         offload.enable = true;
         amdgpuBusId = "PCI:5:0:0";
         nvidiaBusId = "PCI:1:0:0";
       };
     };
   }
   ```

3. Minimal install:  
   ```bash
   nixos-generate-config --root /mnt
   nixos-install --flake github:yourusername/yourrepo#kudu
   ```

**Phase 2: LLM Orchestration Early Integration**
```bash
# Post-install before first reboot
nix-env -iA nixos.claude-code
claude-code setup --automatic --env=system_config
```

**Phase 3: Declarative Environment Build**
1. NVIDIA-optimized Proton:  
   ```nix
   # configuration.nix
   nixpkgs.overlays = [(final: prev: {
     proton-ge = prev.callPackage ./custom/proton-ge.nix {};
   })];
   ```

2. AI/ML stack:  
   ```nix
   environment.systemPackages = with pkgs; [
     cudaPackages.cudatoolkit
     python3.withPackages(ps: [ps.tensorflowWithCuda ps.torchWithCuda])
   ];
   ```

3. Hybrid Graphics Automation:  
   ```bash
   # Create GPU switcher script
   envycontrol -s nvidia --dm lightdm  # For X11 fallback
   ```

**Phase 4: Wayland Environment**
Hyprland configuration example:
```conf
# ~/.config/hypr/hyprland.conf
monitor=,preferred,auto,1
exec-once = dbus-update-activation-environment --systemd WAYLAND_DISPLAY
env = LIBVA_DRIVER_NAME,nvidia
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
```

**Phase 5: Development Environments**
Per-project Nix shells:  
```nix
# shell.nix
{ pkgs ? import <nixpkgs> {} }:
pkgs.mkShell {
  packages = [ pkgs.rustc pkgs.cargo ];
  LD_LIBRARY_PATH = "${pkgs.stdenv.cc.cc.lib}/lib";
}
```

### Critical Considerations
1. **NVIDIA on Wayland**: Use 550+ drivers with explicit modesetting. Patch XWayland for gaming performance:
   ```nix
   nixpkgs.config.packageOverrides = pkgs: {
     xwayland = pkgs.xwayland.overrideAttrs (old: {
       patches = old.patches ++ [ ./xwayland-nvidia.patch ];
     });
   }
   ```

2. **Gentoo Hybrid Compromise**: Set binary packages for:
   ```bash
   # /etc/portage/binrepos.conf
   [gentoo-binhost]
   priority = 9999
   sync-uri = https://gentoo-binary-packages.example.com
   ```
   Use `emerge --getbinpkg` for large packages (LLVM, rust, web browsers).

3. **Music Production Fallback**: Use PipeWire with low-latency kernel when needed:
   ```nix
   # NixOS audio config
   sound.lowLatency = {
     enable = true;
     quantum = 64;
   }
   ```

This approach prioritizes reproducibility, automation compatibility, and gaming/AI performance while maintaining your hardware-specific constraints. The NixOS implementation provides atomic rollbacks - critical when experimenting with bleeding-edge components.
