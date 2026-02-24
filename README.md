# aws-nixos-guide
simple guide for setting up NixOS on AWS EC2

Here is a comprehensive `README.md` guide that documents everything we’ve done to set up your NixOS environment on AWS and connect it to VS Code. You can save this text as a file named `README.md` in your GitHub repository.

---

# NixOS AWS Development Setup Guide

This guide provides a step-by-step walkthrough for launching a NixOS instance on AWS, configuring SSH on a Windows machine, and setting up a professional development environment with NixOS, VS Code, Python, Rust, and Docker.

---

## 1. AWS Instance Launch & Specs

To ensure a smooth experience with modern compilers (like Rust), use the following recommended specifications:

* **AMI:** Search for the official **NixOS** image in the AWS Marketplace or community AMIs.
* **Instance Type:** `t4g.medium` (minimum 4GB RAM is recommended for Rust compilation).
* **Storage:** 20GB+ (Nix stores multiple versions of packages, so space fills up faster than usual).
* **Security Group:** * **Inbound:** Allow **SSH (Port 22)** from your IP.
* **Custom TCP (Port 3000-8080):** Optional, for web development/scraping previews.


* **Key Pair:** Generate a new `.pem` key and download it to your PC.

---

## 2. Windows PC: .pem Key Setup

Windows is strict about SSH key permissions. If the key is "too open," SSH will reject it.

### Fix Permissions (PowerShell)

Run these commands in PowerShell (replace the first line with your actual path):

```powershell
$path = "C:\Users\YourName\Downloads\your-key.pem"
# Reset permissions to default
icacls.exe $path /reset
# Grant Read access to ONLY your user
icacls.exe $path /GRANT:R "$($env:USERNAME):(R)"
# Remove inherited permissions (blocks other users/admins)
icacls.exe $path /inheritance:r

```

---

## 3. Connecting via SSH

You can connect directly via terminal or configure a shortcut.

**Direct Command:**

```bash
ssh -i "C:\path\to\your-key.pem" root@YOUR_AWS_IP

```

**Pro Tip: SSH Config File**
Create a file at `C:\Users\YourName\.ssh\config` and add:

```text
Host nix-box
    HostName YOUR_AWS_IP
    User root
    IdentityFile "C:\path\to\your-key.pem"

```

Now you can simply type `ssh nix-box` to connect.

---

## 4. NixOS Configuration (`configuration.nix`)

NixOS is configured declaratively in `/etc/nixos/configuration.nix`.

### The Essential Setup

To allow VS Code to connect and to install your dev tools, use this configuration:

```nix
{ modulesPath, pkgs, ... }: {
  imports = [ "${modulesPath}/virtualisation/amazon-image.nix" ];
  ec2.efi = true;

  # Enable nix-ld (Required for VS Code Remote SSH)
  programs.nix-ld.enable = true;
  programs.nix-ld.libraries = with pkgs; [
    stdenv.cc.cc zlib fuse3 icu nss openssl curl expat
  ];

  # Enable Docker
  virtualisation.docker.enable = true;

  # Install Development Tools
  environment.systemPackages = with pkgs; [
    git
    python3
    python3Packages.pip
    rustc
    cargo
    gcc
    pkg-config
    chromium # For web scraping
  ];

  # Security: Disable root password login (Keep .pem login)
  services.openssh.settings.PermitRootLogin = "prohibit-password";
}

```

### Applying Changes

After saving the file, apply the changes by running:

```bash
nixos-rebuild switch

```

---

## 5. VS Code Integration

1. Install the **Remote - SSH** extension in VS Code.
2. Press `F1` -> **Remote-SSH: Connect to Host...** -> Select your `nix-box`.
3. Once connected, if you see "Fetching remote environment" errors, ensure `nix-ld` is enabled and you have run `nixos-rebuild switch`.
4. **To view web results:** VS Code will automatically forward ports (like `3000`) so you can view the server's web pages in your Windows browser at `http://localhost:3000`.

---

[NixOS on AWS installation guide](https://www.youtube.com/watch?v=Ee4JN3Fp17o)

This video provides a deep dive into scalable and secure NixOS deployments on AWS, helping you understand how to manage your instance as your projects grow.
