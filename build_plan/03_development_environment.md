# 3. Development Environment

This section declaratively defines the development environment. We will create a separate Nix file (`dev.nix`) to keep our main `configuration.nix` clean and import it.

All changes should be made by editing the files in `/etc/nixos/` and applying them with `sudo nixos-rebuild switch --flake .#gimli`.

### Step 3.1: Create `dev.nix`

1.  Create a new file named `dev.nix` in `/etc/nixos/`.
    ```bash
    sudo nano /etc/nixos/dev.nix
    ```
2.  Paste the following content. This includes the requested programming languages, toolchains, and containerization software.

    ```nix
    # /etc/nixos/dev.nix
    { config, pkgs, ... }:

    {
      # Add all development-related packages here
      environment.systemPackages = with pkgs; [
        # --- Languages & Toolchains ---
        # C/C++
        gcc
        clang
        cmake
        gdb

        # Python
        python3
        python3Packages.pip

        # Java (OpenJDK 21)
        jdk21

        # Rust
        rustup

        # Go
        go

        # Node.js
        nodejs

        # Haskell
        ghc
        cabal-install

        # Lisp (SBCL)
        sbcl

        # Lua
        lua

        # Perl
        perl

        # --- Cloud SDKs ---
        google-cloud-sdk
        awscli2
        azure-cli

        # --- Containerization ---
        docker
        kubectl
        kind
      ];

      # --- Service Configuration ---

      # Enable Docker daemon
      virtualisation.docker.enable = true;
      # Add user to the docker group
      users.users.chris.extraGroups = [ "docker" ];

      # Install Rust toolchain
      # This is a post-activation command that runs `rustup` for the user.
      system.activationScripts.rustup_init = ''
        runuser -l chris -c 'rustup-init -y'
      '';
    }
    ```

### Step 3.2: Import `dev.nix`

Now, import the new file from your `configuration.nix`.

1.  Edit your main configuration file:
    ```bash
    sudo nano /etc/nixos/configuration.nix
    ```
2.  Add `./dev.nix` to the `imports` list:
    ```nix
    # configuration.nix
    { config, pkgs, ... }:

    {
      imports = [
        ./hardware-configuration.nix
        ./dev.nix  # <-- Add this line
      ];

      # ... rest of your configuration
    }
    ```

### Step 3.3: Rebuild and Verify

1.  Apply the new configuration:
    ```bash
    sudo nixos-rebuild switch --flake .#gimli
    ```
2.  After the rebuild, your system will have all the specified development tools.
3.  Commit the changes:
    ```bash
    cd /etc/nixos/
    sudo git add .
    sudo git commit -m "feat: add development environment"
    ```
4.  Verify some of the installations:
    ```bash
    python3 --version
    go version
    docker --version
    ```

Your system is now equipped with a comprehensive and declaratively managed development environment.
