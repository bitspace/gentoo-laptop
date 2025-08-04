# 2. Core System and Flake Configuration

With the base system installed, we will now structure the configuration using **Nix Flakes**. This is the foundation for our declarative system. From this point on, all changes will be made by editing configuration files and running `nixos-rebuild switch`.

Log in as `root` at the TTY prompt.

### Step 2.1: Connect to the Network

After rebooting from the installation, you should be at a TTY login prompt. Log in as `root` with the password you set during the install.

Because you enabled `networking.networkmanager.enable = true;` in the previous step, the NetworkManager service is running. You now need to use it to connect to your Wi-Fi network.

1.  **Start the NetworkManager TUI:**
    ```bash
    nmtui
    ```
2.  **Connect to Your Network:**
    - A text-based user interface will open.
    - Select **"Activate a connection"** and press Enter.
    - Select your Wi-Fi network (SSID) from the list and press Enter.
    - Enter your Wi-Fi password when prompted and press Enter.
    - Once it connects, select **"Back"** and then **"Quit"**.
3.  **Verify the Connection:**
    ```bash
    ping nixos.org
    ```
    If you get a response, you are online. NetworkManager will now remember this connection and connect automatically on future boots.

### Step 2.2: Create the Flake Structure

1.  Navigate to your NixOS configuration directory:
    ```bash
    cd /etc/nixos/
    ```
2.  This directory should contain `configuration.nix` and `hardware-configuration.nix`. We are going to create a new `flake.nix` file.
3.  Enable git and make an initial commit to track our changes.
    ```bash
    # You may need to install git first
    nix-env -iA nixos.git
    # Configure git with a placeholder identity. This is required to make commits.
    git config --global user.name "NixOS User"
    git config --global user.email "root@localhost"
    git init
    git add .
    git commit -m "Initial NixOS configuration"
    ```
4.  Create the `flake.nix` file by running the following command. It will write the entire contents of the file at once.
    ```bash
    cat > flake.nix <<'EOF'
    {
      description = "A declarative NixOS system for Gimli";

      inputs = {
        nixpkgs.url = "github:nixos/nixpkgs/nixos-unstable";
        home-manager.url = "github:nix-community/home-manager";
        home-manager.inputs.nixpkgs.follows = "nixpkgs";
      };

      outputs = { self, nixpkgs, home-manager, ... }@inputs: {
        nixosConfigurations.gimli = nixpkgs.lib.nixosSystem {
          system = "x86_64-linux";
          specialArgs = { inherit inputs; };
          modules = [
            ./configuration.nix
            home-manager.nixosModules.home-manager
            {
              home-manager.useGlobalPkgs = true;
              home-manager.useUserPackages = true;
              home-manager.extraSpecialArgs = { inherit inputs; };
            }
          ];
        };
      };
    }
    EOF
    ```


### Step 2.3: Refactor `configuration.nix`

Now we need to adjust our main `configuration.nix` to work within the flake. The following commands will do this for you automatically.

1.  **Preserve your unique `system.stateVersion`**: This command finds the state version line in your current configuration and saves it to a shell variable.
    ```bash
    STATE_VERSION_LINE=$(grep 'system.stateVersion' /etc/nixos/configuration.nix)
    ```

2.  **Generate the new configuration**: This command overwrites your `configuration.nix` with the new content, automatically inserting your specific `system.stateVersion` at the end.
    ```bash
    cat > configuration.nix <<EOF
    { config, pkgs, ... }:

    {
      imports = [ ./hardware-configuration.nix ];

      # Bootloader
      boot.loader.systemd-boot.enable = true;
      boot.loader.efi.canTouchEfiVariables = true;

      # Networking
      networking.hostName = "gimli"; # Define your hostname.
      networking.networkmanager.enable = true;

      # Set your time zone.
      time.timeZone = "America/New_York"; # e.g. "Europe/Berlin"

      # Select internationalisation properties.
      i18n.defaultLocale = "en_US.UTF-8";

      # Configure keymap in X11
      services.xserver = {
        layout = "us";
        xkbVariant = "";
      };

      # Define a user account. Don't forget to set a password with `passwd`.
      users.users.chris = {
        isNormalUser = true;
        description = "Chris";
        extraGroups = [ "wheel" "networkmanager" ];
      };

      # Allow unfree packages
      nixpkgs.config.allowUnfree = true;

      # List packages installed in system profile.
      environment.systemPackages = with pkgs;
        [ vim git wget curl htop
          # Add your Gemini CLI package here later
        ];

      # Enable the OpenSSH daemon.
      services.openssh.enable = true;

      # This value determines the NixOS release from which the default
      # settings for stateful data, like file locations and database versions
      # on your system were taken. It‘s perfectly fine and recommended to leave
      # this value at the release version of the first install of this system.
      # Before changing this value read the documentation for this option
      # (e.g. man configuration.nix or on https://nixos.org/nixos/options.html).
      ${STATE_VERSION_LINE}
    }
    EOF
    ```

### Step 2.3: First Declarative Rebuild

Now we apply our new flake-based configuration for the first time.

1.  Run the rebuild command. Note that we are referencing the flake (`.#gimli`) in the current directory.
    ```bash
    nixos-rebuild switch --flake .#gimli
    ```
2.  This will download the new flake inputs (`home-manager`) and rebuild your system.
3.  After it succeeds, create your user's password:
    ```bash
    passwd chris
    ```
4.  Commit the changes:
    ```bash
    git add .
    git commit -m "feat: Convert to flake and add user"
    ```

### Step 2.4: Gemini CLI Integration

At this point, you would add the Gemini CLI to your system. Assuming it's available as a Nix package (e.g., `pkgs.gemini-cli`), you would add it to the `environment.systemPackages` list in `configuration.nix` and rebuild.

If it's not packaged, you would create a Nix package for it, but for now, we will assume it can be added to the list.

**From now on, you should log in as `chris` and use `sudo` for commands that require root privileges.** The rest of this guide will assume you are operating as the `chris` user.
