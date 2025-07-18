# CXB3RF1LTHL4BZ Virtual Machine Lab Installer

Two high-performance, fully automated scripts to set up a complete virtualization environment for:

* Debian/Ubuntu-based systems
* Arch Linux / Hyprland setups

Built for red teams, malware labs, offensive security labs, and virtual lab enthusiasts who don't have time for broken configs or missing dependencies.

---

## Features

* Fully automated
* Sets up VirtualBox, QEMU/KVM, Libvirt, SPICE, and virt-manager
* Configures default NAT network
* Handles user group permissions
* Fixes known conflicts (e.g., iptables on Arch)
  

---

## Requirements

* Root privileges (sudo)
* Internet access
* A no-nonsense approach

---

## Debian/Ubuntu Installation

### Save Script

```bash
nano cxb3rf1lthl4bz-debvirt-installer.sh
```

Paste the Debian script content inside.

### Run Script

```bash
chmod +x cxb3rf1lthl4bz-debvirt-installer.sh
sudo ./cxb3rf1lthl4bz-debvirt-installer.sh
```

---

## Arch Linux / Hyprland Installation

### Save Script

```bash
nano cxb3rf1lthl4bz_vm_env_installer.sh
```

Paste the Arch script content inside.

### Run Script

```bash
chmod +x cxb3rf1lthl4bz_vm_env_installer.sh
sudo ./cxb3rf1lthl4bz_vm_env_installer.sh
```

---

## Post-Install

Reboot your machine to ensure group changes and services take effect:

```bash
sudo reboot
```

Then launch virt-manager or virtualbox and start building your lab.

---

## Scripts

### Debian Script

<details>
<summary>Click to expand</summary>

```bash
#!/bin/bash
# CXB3RF1LTHL4BZ-bLACKcELL VM Installer for Debian

set -euo pipefail

log() { echo -e "\e[1;36m[*]\e[0m $1"; }
err() { echo -e "\e[1;31m[!] ERROR:\e[0m $1" >&2; exit 1; }

check_root() {
  [[ "$EUID" -ne 0 ]] && err "Please run as root (sudo)"
}

main() {
  check_root
  clear
  echo -e "\n\e[1;35mCXB3RF1LTHL4BZ-bLACKcELL Virtual Machine Lab Installer for Debian\e[0m\n"

  log "Updating system..."
  apt update && apt full-upgrade -y || err "System update failed"

  log "Installing virtualization stack..."
  apt install -y virtualbox virtualbox-ext-pack qemu-kvm libvirt-daemon-system \
    libvirt-clients virt-manager bridge-utils dnsmasq-base ovmf spice-client-gtk \
    virt-viewer netcat-openbsd wget curl unzip || err "Failed installing virtualization stack"

  log "Adding user to groups..."
  usermod -aG libvirt,kvm $(logname) || err "Group add failed"

  log "Enabling libvirt services..."
  systemctl enable --now libvirtd || err "Failed to enable libvirtd"

  log "Ensuring default NAT network..."
  virsh net-list --all | grep -q default || virsh net-define /usr/share/libvirt/networks/default.xml || true
  virsh net-autostart default || true
  virsh net-start default || true

  log "Done. Reboot recommended."
}

main "$@"
```

</details>

### Arch Script

<details>
<summary>Click to expand</summary>

```bash
#!/bin/bash
# CXB3RF1LTHL4BZ-bLACKcELL VM Installer for Arch

set -euo pipefail

log() { echo -e "\e[1;36m[*]\e[0m $1"; }
err() { echo -e "\e[1;31m[!] ERROR:\e[0m $1" >&2; exit 1; }

check_root() {
  [[ "$EUID" -ne 0 ]] && err "Please run as root (sudo)"
}

fix_conflicts() {
  if pacman -Q iptables &>/dev/null && pacman -Q iptables-nft &>/dev/null; then
    log "Removing conflicting iptables..."
    pacman -Rdd --noconfirm iptables || err "Failed to remove iptables"
  fi
}

main() {
  check_root
  clear
  echo -e "\n\e[1;35mCXB3RF1LTHL4BZ-bLACKcELL Virtual Machine Lab Installer for Arch Linux\e[0m\n"

  log "Fixing conflicts..."
  fix_conflicts

  log "Updating system..."
  pacman -Syu --noconfirm || err "System update failed"

  log "Installing virtualization stack..."
  pacman -S --noconfirm virtualbox virtualbox-host-modules-arch virtualbox-guest-iso \
    qemu-base virt-manager libvirt dnsmasq bridge-utils edk2-ovmf spice spice-gtk \
    virt-viewer openbsd-netcat wget curl unzip || err "Install failed"

  log "Enabling libvirt..."
  systemctl enable --now libvirtd.service || err "Enable failed"

  log "Adding user to groups..."
  usermod -aG libvirt,kvm $(logname) || err "Group add failed"

  log "Configuring default network..."
  virsh net-define /usr/share/libvirt/networks/default.xml 2>/dev/null || true
  virsh net-autostart default 2>/dev/null || true
  virsh net-start default 2>/dev/null || true

  log "Done. Reboot recommended."
}

main "$@"
```

</details>

---

## Credits

Crafted by Cxb3rf1lthl4bz 

Tested on:

* Kali Linux 2024.X
* Arch Linux (Linux-zen kernel, Hyprland)

VM-friendly with QEMU and VirtualBox.

---

## Pro Tips

* Want GPU passthrough? Customize the script with VFIO modules.
* Prefer UEFI-only guests? Use ovmf (already installed).
* Use virt-manager + SPICE for smooth GUI guests.
* Set up NAT + Bridge if you're simulating advanced malware or C2 infrastructure.

---

Keep it virtual. Keep it legal. Keep it sharp.

