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
    - **For Wi-Fi:** The minimal installer uses `wpa_supplicant`.
      1.  Find your wireless interface name (it usually starts with `w`, like `wlan0` or `wlp2s0`):
          ```bash
          ip link
          ```
      2.  Generate a configuration for your network. Replace `"Your-SSID"` and `"Your-Password"` with your actual Wi-Fi name and password. This command will create a configuration file in your current directory.
          ```bash
          wpa_passphrase "Your-SSID" "Your-Password" > wpa_supplicant.conf
          ```
      3.  Start the Wi-Fi connection in the background. Replace `<device>` with your interface name from step 1.
          ```bash
          sudo wpa_supplicant -B -i <device> -c wpa_supplicant.conf
          ```
      4.  Get an IP address from the network.
          ```bash
          sudo dhcpcd
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

### Step 1.5.1: Configure Persistent Networking

For the network connection to work after you reboot into your new system, you must enable a network management service in `configuration.nix`. The `wpa_supplicant.conf` file you created earlier was only to get online in the temporary live environment.

The standard and most flexible tool for this is NetworkManager.

1.  **Edit the configuration file** for your new system:
    ```bash
    sudo nano /mnt/etc/nixos/configuration.nix
    ```

2.  **Enable NetworkManager:** Add the following line inside the main `{ ... }` section of the file. A good place is near the `services.openssh.enable` line.
    ```nix
    # Enable the NetworkManager service.
    networking.networkmanager.enable = true;
    ```

3.  Save and exit the editor (`Ctrl+O`, `Enter`, `Ctrl+X` in nano).

That's it. When you later boot into your installed system, the NetworkManager service will be running. You will use its command-line interface, `nmtui`, to connect to your Wi-Fi the first time, and it will automatically manage the connection from then on.


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
