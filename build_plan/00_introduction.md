# Build Plan: Declarative NixOS Workstation

## 0. Introduction

This build plan provides a step-by-step guide to install and configure a fully declarative NixOS system running the Hyprland Wayland compositor. The entire process is designed to be automated and managed via configuration files, aligning with the project's goal of an LLM-orchestrated environment.

### Philosophy

The guiding principle is **"Configuration as Code."** We will perform a minimal manual installation and then define the entire system—from hardware drivers to development tools and gaming clients—within a set of version-controlled Nix configuration files (`flakes`).

This approach provides:
- **Reproducibility:** The ability to recreate the exact same system on any compatible hardware.
- **Atomicity:** System updates either succeed completely or not at all, eliminating broken states.
- **Rollbacks:** Every configuration change creates a new "generation" that you can instantly roll back to.

### Structure of the Plan

The plan is broken down into numbered stages, corresponding to the files in this directory.

- **01_base_install.md:** The initial, manual steps to get a minimal NixOS system running. This is the only phase that requires significant interactive commands.
- **02_core_system_and_cli.md:** Creating the core declarative flake structure and configuring users, networking, and SSH.
- **03_development_environment.md:** Declaratively adding all required programming languages, toolchains, and container services.
- **04_desktop_environment.md:** Installing and configuring the Hyprland compositor, Wayland, and core desktop utilities like Pipewire for audio.
- **05_gaming_environment.md:** Installing Steam, Proton, and other gaming-related software.
- **06_system_theming.md:** Applying themes, fonts, and other quality-of-life customizations.
- **99_validation_and_next_steps.md:** Verifying the installation and outlining how to manage the system going forward.

Let's begin with the base installation.
