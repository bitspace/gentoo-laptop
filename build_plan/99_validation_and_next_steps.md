# 99. Validation and Next Steps

Congratulations! You have successfully built a fully declarative NixOS system. This document outlines how to validate the installation and how to manage your system going forward.

### Step 99.1: Validation Checklist

Run these commands in a terminal to verify that the core components are working as expected.

1.  **Check GPU Drivers:**
    ```bash
    # Should show both AMD and NVIDIA devices
    lspci | grep -E 'VGA|3D'

    # Should show NVIDIA driver details and processes
    nvidia-smi

    # To confirm PRIME offload is working
    prime-run glxinfo | grep "OpenGL renderer"
    # The output should say "NVIDIA"
    ```

2.  **Check Wayland Session:**
    ```bash
    # Should show 'wayland'
    echo $XDG_SESSION_TYPE
    ```

3.  **Check Audio:**
    -   Open `pavucontrol` from the application launcher.
    -   Play some audio (e.g., from a web browser) and verify that you see activity in the "Playback" tab and can control the volume.

4.  **Check Development Tools:**
    ```bash
    # Check a few key tools
    python3 --version
    docker info
    rustc --version
    ```

5.  **Check Gaming:**
    -   Launch Steam.
    -   Ensure you can log in and that it sees your game library.
    -   Check the "Steam Play" settings to confirm that Proton-GE is available as a compatibility tool.

### Step 99.2: Managing Your System

Your system is now managed entirely through the Nix configuration files in `/etc/nixos/`. Here is the standard workflow for making changes:

1.  **Edit:** Make your desired changes to the `.nix` files (e.g., add a package to `environment.systemPackages`, change a Hyprland keybinding in `home.nix`, etc.).

2.  **Rebuild:** Apply the changes with the rebuild command:
    ```bash
    sudo nixos-rebuild switch --flake /etc/nixos/#gimli
    ```

3.  **Commit:** After verifying the change works, commit it to your git repository. This creates a historical record and a logical checkpoint.
    ```bash
    cd /etc/nixos/
    sudo git add .
    sudo git commit -m "feat: Added xyz package"
    ```

### Step 99.3: Rollbacks

If a change causes problems, rolling back is trivial.

-   **To temporarily roll back:** At the bootloader menu, you will see a list of all your previous NixOS "generations." Simply select an older one to boot into the system as it was before the change.
-   **To permanently roll back:** Run the rollback command:
    ```bash
    sudo nixos-rebuild switch --rollback
    ```
    This will make the previous generation the new default. You can then fix the configuration and rebuild forward.

### Next Steps

Your journey with declarative systems is just beginning. Here are some ideas for what to explore next:

-   **Explore Home Manager:** Dive deeper into the options Home Manager provides for managing your dotfiles. You can configure almost any application with it.
-   **Create Development Shells:** For your programming projects, create `flake.nix` files in your project directories to define exact, per-project dependencies. This is done with `pkgs.mkShell`.
-   **Secrets Management:** Look into tools like `agenix` or `sops-nix` for managing secrets (API keys, passwords) declaratively and securely.
-   **Custom Packages:** Learn how to package your own software or override existing packages in Nix.

Welcome to the future of system administration!
