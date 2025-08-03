# 6. System Theming and Dotfile Management

Now that all the system-level packages are installed, we will configure the user-specific "dotfiles" for Hyprland, Waybar, Alacritty, etc. We will use **Home Manager** to do this declaratively, just like we did for the system.

This approach keeps your user-level configuration in Nix, version-controlled, and reproducible.

### Step 6.1: Create a Home Manager Configuration

1.  We need to create a directory for our user's configuration.
    ```bash
    sudo mkdir -p /etc/nixos/home/chris
    ```
2.  Create a `home.nix` file inside that directory. This will be the main entry point for `chris`'s configuration.
    ```bash
    sudo nano /etc/nixos/home/chris/home.nix
    ```
3.  Paste the following content. This sets up a basic, functional Hyprland configuration.

    ```nix
    # /etc/nixos/home/chris/home.nix
    { config, pkgs, ... }:

    {
      # Home Manager needs a bit of information about you and the paths it should
      # manage.
      home.username = "chris";
      home.homeDirectory = "/home/chris";

      # This value determines the Home Manager release that your configuration is
      # compatible with. This helps avoid breakage when a new Home Manager release
      # introduces backwards incompatible changes.
      home.stateVersion = "23.11";

      # Let Home Manager manage itself
      programs.home-manager.enable = true;

      # --- Add user-level packages here ---
      home.packages = with pkgs; [
        # Add any user-specific packages here
      ];

      # --- Hyprland Configuration ---
      wayland.windowManager.hyprland = {
        enable = true;
        settings = {
          # Monitors
          monitor = ",preferred,auto,1";

          # Keybindings
          bind = [
            "SUPER, RETURN, exec, alacritty"
            "SUPER, Q, killactive,"
            "SUPER, M, exit,"
            "SUPER, E, exec, dolphin" # Assuming you install dolphin
            "SUPER, D, exec, wofi --show drun"
            "SUPER, P, pseudo," # dwindle
            "SUPER, J, togglesplit," # dwindle

            # Move focus with mainMod + arrow keys
            "SUPER, left, movefocus, l"
            "SUPER, right, movefocus, r"
            "SUPER, up, movefocus, u"
            "SUPER, down, movefocus, d"

            # Switch workspaces with mainMod + [0-9]
            "SUPER, 1, workspace, 1"
            "SUPER, 2, workspace, 2"
            "SUPER, 3, workspace, 3"

            # Move active window to a workspace with mainMod + SHIFT + [0-9]
            "SUPER SHIFT, 1, movetoworkspace, 1"
            "SUPER SHIFT, 2, movetoworkspace, 2"
            "SUPER SHIFT, 3, movetoworkspace, 3"
          ];

          # Startup applications
          exec-once = [
            "waybar"
            "mako"
          ];
        };
      };

      # --- Waybar Configuration ---
      programs.waybar = {
        enable = true;
        settings = {
          mainBar = {
            layer = "top";
            position = "top";
            modules-left = [ "sway/workspaces" "sway/mode" ];
            modules-center = [ "sway/window" ];
            modules-right = [ "pulseaudio" "network" "clock" ];
            "sway/window" = {
              "max-length" = 25;
            };
          };
        };
      };

      # --- Alacritty Terminal Configuration ---
      programs.alacritty = {
        enable = true;
        settings = {
          font = {
            normal.family = "JetBrainsMono Nerd Font";
            size = 12;
          };
        };
      };
    }
    ```

### Step 6.2: Link the Home Manager Configuration

Now, tell NixOS to use this configuration for the `chris` user.

1.  Edit your main `configuration.nix`:
    ```bash
    sudo nano /etc/nixos/configuration.nix
    ```
2.  Add the `home-manager.users.chris` option:
    ```nix
    # configuration.nix
    # ... inside the main { }

    home-manager.users.chris = import ./home/chris/home.nix;

    # ... rest of your configuration
    ```

### Step 6.3: Rebuild and See the Magic

1.  Apply the configuration:
    ```bash
    sudo nixos-rebuild switch --flake .#gimli
    ```
2.  Commit the changes:
    ```bash
    cd /etc/nixos/
    sudo git add .
    sudo git commit -m "feat: add home-manager config for hyprland"
    ```
3.  The changes should apply instantly. You should see Waybar appear at the top of your screen, and your keybindings (`Super+Enter` for Alacritty) should now work. If they don't, a reboot might be necessary for all services to pick up the changes.

You now have a fully declarative system, from the kernel and drivers all the way up to your personal application settings.
