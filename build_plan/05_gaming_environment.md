# 5. Gaming Environment

This section adds the necessary components for gaming, including Steam, Wine, and Proton. We will create a `gaming.nix` file and import it.

### Step 5.1: Create `gaming.nix`

1.  Create the new file:
    ```bash
    sudo nano /etc/nixos/gaming.nix
    ```
2.  Paste the following configuration:

    ```nix
    # /etc/nixos/gaming.nix
    { config, pkgs, ... }:

    {
      # Add gaming-related packages
      environment.systemPackages = with pkgs; [
        # --- Clients & Launchers ---
        steam
        lutris
        wineWowPackages.stable # For running Windows apps outside of Steam

        # --- Compatibility & Tools ---
        proton-ge-bin # GloriousEggroll's custom Proton build
        winetricks
        gamemode
        mangohud
      ];

      # --- Steam Configuration ---
      programs.steam = {
        enable = true;
        # Add Proton-GE to the list of available compatibility tools
        extraCompatPackages = [ pkgs.proton-ge-bin ];
        # Open firewall for remote play
        remotePlay.openFirewall = true;
        # For games that use VAC
        dedicatedServer.openFirewall = true;
      };

      # Enable 32-bit support, which is crucial for many games
      hardware.opengl.driSupport32Bit = true;
    }
    ```

### Step 5.2: Import `gaming.nix`

1.  Add the new file to your `configuration.nix` imports:
    ```bash
    sudo nano /etc/nixos/configuration.nix
    ```
    ```nix
    # configuration.nix
    imports = [
      ./hardware-configuration.nix
      ./dev.nix
      ./desktop.nix
      ./gaming.nix # <-- Add this line
    ];
    ```

### Step 5.3: Rebuild and Verify

1.  Apply the configuration:
    ```bash
    sudo nixos-rebuild switch --flake .#gimli
    ```
2.  Commit the changes:
    ```bash
    cd /etc/nixos/
    sudo git add .
    sudo git commit -m "feat: add gaming environment"
    ```
3.  You can now launch Steam from the application launcher (e.g., by pressing `Super+D` and typing `steam`).

### A Note on Performance

To launch a game on the dedicated NVIDIA GPU, you can use the `prime-run` command. Steam should do this automatically for Vulkan games. For other applications or games, you can launch them from the terminal:

```bash
# This will run the command on the NVIDIA GPU
prime-run glxinfo
```

You can also use `gamemoderun` to apply performance tweaks:

```bash
gamemoderun prime-run your-game-command
```
