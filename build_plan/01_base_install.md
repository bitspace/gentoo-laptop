# 1. Base System Installation

This phase covers the manual steps required to get a minimal, bootable NixOS system on your machine. This is the most "imperative" part of the process.

### Step 1.1: Prepare Installation Media

1.  Download the latest **NixOS Minimal ISO** from the [official website](https://nixos.org/download.html).
2.  Create a bootable USB drive using a tool like `dd` or Balena Etcher.
    ```bash
    # Example using dd. Replace /dev/sdX with your USB device.
    sudo dd if=/path/to/nixos-minimal-*.iso of=/dev/sdX bs=4M conv=fsync status=progress
    ```

### Step 1.2: Boot and Network Connection

1.  Boot your System76 Kudu from the USB drive. You should be greeted by a shell prompt.
2.  Connect to your network.
    - **For Ethernet:** It should connect automatically.
    - **For Wi-Fi:** Use the `iwctl` utility.
      ```bash
      # Enter the interactive iwctl prompt
      iwctl
      # List your Wi-Fi devices (e.g., wlan0)
      [iwd]# device list
      # Scan for networks
      [iwd]# station <device> scan
      # List available networks
      [iwd]# station <device> get-networks
      # Connect to your network
      [iwd]# station <device> connect "Your-SSID" --passphrase "Your-Password"
      # Exit iwctl
      [iwd]# exit
      ```
3.  Verify your connection:
    ```bash
    ping nixos.org
    ```

### Step 1.3: Partitioning

We will use `gdisk` to partition the NVMe drive according to your specified layout.

1.  Start `gdisk` on your target drive:
    ```bash
    sudo gdisk /dev/nvme0n1
    ```
2.  Inside `gdisk`, create the following partitions. Delete any existing partitions first if necessary (using the `d` command).
    - **Partition 1 (EFI):**
        - `n` (new partition)
        - Partition number: `1`
        - First sector: (default)
        - Last sector: `+1G`
        - Hex code: `EF00` (EFI System)
    - **Partition 2 (Swap):**
        - `n` (new partition)
        - Partition number: `2`
        - First sector: (default)
        - Last sector: `+16G`
        - Hex code: `8200` (Linux swap)
    - **Partition 3 (Root):**
        - `n` (new partition)
        - Partition number: `3`
        - First sector: (default)
        - Last sector: (default, to use remaining space)
        - Hex code: `8300` (Linux filesystem)
3.  Write changes to disk with `w` and confirm.

### Step 1.4: Formatting and Mounting

1.  Format the filesystems:
    ```bash
    sudo mkfs.fat -F 32 -n BOOT /dev/nvme0n1p1
    sudo mkswap -L SWAP /dev/nvme0n1p2
    sudo mkfs.xfs -L ROOT /dev/nvme0n1p3
    ```
2.  Mount the filesystems:
    ```bash
    sudo mount /dev/disk/by-label/ROOT /mnt
    sudo mkdir -p /mnt/boot
    sudo mount /dev/disk/by-label/BOOT /mnt/boot
    sudo swapon /dev/nvme0n1p2
    ```

### Step 1.5: Generate Initial NixOS Configuration

1.  Generate the base NixOS configuration files:
    ```bash
    sudo nixos-generate-config --root /mnt
    ```
2.  This creates `configuration.nix` and `hardware-configuration.nix` in `/mnt/etc/nixos/`. We will edit these in the next phase.
3.  For now, enable flakes and SSH daemon in the generated configuration so you can manage the system remotely after the first boot.
    ```bash
    # Use nano or vim to edit the file
    sudo nano /mnt/etc/nixos/configuration.nix
    ```
    Add the following lines inside the `{ ... }`:
    ```nix
    # Enable flakes
    nix.settings.experimental-features = [ "nix-command" "flakes" ];

    # Enable SSH daemon
    services.openssh.enable = true;
    ```

### Step 1.6: Install and Reboot

1.  Run the NixOS installation:
    ```bash
    sudo nixos-install
    ```
2.  This process will take some time. Once it's complete, it will prompt you to set a root password. **Set a strong password.**
3.  Reboot the system:
    ```bash
    sudo reboot
    ```

Remove the USB drive when the system powers down. On the next boot, you should be greeted by the NixOS bootloader and then a TTY login prompt.

**You have now completed the manual base installation.** In the next phase, we will structure the configuration using flakes and start defining the system declaratively.
