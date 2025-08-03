# 4. Desktop Environment (Hyprland)

This section details how to declaratively install and configure the Hyprland Wayland compositor, along with essential components for a modern desktop experience like PipeWire for audio and Polkit for permissions.

We will follow the same pattern of creating a dedicated file (`desktop.nix`) and importing it.

### Step 4.1: Create `desktop.nix`

1.  Create the new file:
    ```bash
    sudo nano /etc/nixos/desktop.nix
    ```
2.  Paste the following configuration. This is a comprehensive setup for Hyprland.

    ```nix
    # /etc/nixos/desktop.nix
    { config, pkgs, ... }:

    {
      # Enable the Hyprland Wayland compositor
      programs.hyprland = {
        enable = true;
        xwayland.enable = true; # For X11 compatibility
      };

      # Enable sound with PipeWire
      sound.enable = true;
      hardware.pulseaudio.enable = false; # Ensure pulseaudio is not used
      security.rtkit.enable = true;
      services.pipewire = {
        enable = true;
        alsa.enable = true;
        alsa.support32Bit = true;
        pulse.enable = true;
        # If you want to use JACK applications, uncomment the following
        #jack.enable = true;
      };

      # Enable Polkit for authentication
      security.polkit.enable = true;

      # Required for screen sharing and other portals
      xdg.portal = {
        enable = true;
        extraPortals = [ pkgs.xdg-desktop-portal-hyprland ];
      };

      # Install essential desktop packages
      environment.systemPackages = with pkgs; [
        # --- Terminal ---
        alacritty

        # --- App Launcher & Bar ---
        waybar
        wofi

        # --- Notifications ---
        mako

        # --- Screenshot Tool ---
        grim
        slurp

        # --- Other Essentials ---
        wl-clipboard # Wayland clipboard utilities
        cliphist      # Clipboard history
        pavucontrol   # Volume control GUI
        network-manager-applet # For managing network in waybar
      ];

      # --- Font Configuration ---
      fonts.packages = with pkgs; [
        noto-fonts
        noto-fonts-cjk
        noto-fonts-emoji
        font-awesome
        (nerdfonts.override { fonts = [ "JetBrainsMono" ]; })
      ];

      # --- Auto-login to TTY and start Hyprland ---
      # This provides a seamless boot-to-desktop experience
      services.getty.autologinUser = "chris";
      programs.bash.loginShellInit = ''
        if [[ "$(tty)" == "/dev/tty1" ]]; then
          exec Hyprland
        fi
      '';
    }
    ```

### Step 4.2: Import `desktop.nix`

1.  Add the new file to your `configuration.nix` imports:
    ```bash
    sudo nano /etc/nixos/configuration.nix
    ```
    ```nix
    # configuration.nix
    imports = [
      ./hardware-configuration.nix
      ./dev.nix
      ./desktop.nix # <-- Add this line
    ];
    ```

### Step 4.3: Configure NVIDIA Drivers

This is a critical step for your hardware. We will add the NVIDIA driver configuration directly to `configuration.nix`.

1.  Edit your main configuration file and add the following block. You may need to find the correct PCI bus IDs for your specific hardware using `lspci`.
    ```bash
    sudo nano /etc/nixos/configuration.nix
    ```
    ```nix
    # Add this block to your configuration.nix

    # NVIDIA specific configuration
    hardware.opengl = {
      enable = true;
      driSupport = true;
      driSupport32Bit = true;
    };

    services.xserver.videoDrivers = ["nvidia"];

    hardware.nvidia = {
      modesetting.enable = true;
      powerManagement.enable = true;
      # Open source driver is not recommended for gaming/CUDA
      open = false;
      nvidiaSettings = true;

      # This is the important part for your hybrid graphics laptop
      prime = {
        sync.enable = true;
        # Find these values with `lspci | grep -E 'VGA|3D'`
        # The first is your AMD iGPU, the second is your NVIDIA dGPU
        amdgpuBusId = "PCI:5:0:0";
        nvidiaBusId = "PCI:1:0:0";
      };
    };
    ```

### Step 4.4: Rebuild and First Graphical Boot

1.  Apply the configuration:
    ```bash
    sudo nixos-rebuild switch --flake .#gimli
    ```
2.  Commit the changes:
    ```bash
    cd /etc/nixos/
    sudo git add .
    sudo git commit -m "feat: add hyprland desktop environment"
    ```
3.  Reboot the system:
    ```bash
    sudo reboot
    ```

Your system should now automatically log you in and start Hyprland. You will see a default, unconfigured desktop. We will add the configuration for Hyprland and Waybar in the theming section using `home-manager`.
