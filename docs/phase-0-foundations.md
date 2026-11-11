# Phase 0: Foundations

Status: complete (11 Nov 2026)

## Goal

A Rocky Linux 9 VM on the MacBook, reachable by SSH key only, with a fixed IP, a minimal firewall, SELinux enforcing and LVM storage configured by hand.

## Environment

| Item | Value |
|---|---|
| Host | MacBook Pro M2, 8 GB RAM, macOS 26 |
| Hypervisor | UTM (QEMU backend) |
| Guest | Rocky Linux 9.8 minimal, aarch64 |
| VM size | 2 GB RAM, 2 vCPU, 20 GB system disk, 2 GB data disk |
| Hostname / IP | rocky-01 / 192.168.64.10 |

## Steps

### 1. Install

- Used the Arm (aarch64) minimal ISO; x86 images run under emulation and are too slow on Apple silicon.
- In the installer: kdump off to save memory, admin user created, root password set, root SSH login off.
- After install, stop the VM and clear the ISO from the CD/DVD drive, or the installer starts again.

### 2. SSH key access (from the Mac)

~~~
ssh-keygen -t ed25519 -C "yusuf-homelab"
ssh-copy-id yusuf@192.168.64.2
~~~

Added a `~/.ssh/config` entry so `ssh rocky-01` works.

### 3. Static IP

~~~
sudo nmcli con mod enp0s1 ipv4.method manual ipv4.addresses 192.168.64.10/24 ipv4.gateway 192.168.64.1 ipv4.dns 192.168.64.1
sudo nmcli con up enp0s1
~~~

### 4. SSH hardening

`/etc/ssh/sshd_config.d/10-hardening.conf`:

~~~
PasswordAuthentication no
PermitRootLogin no
~~~

Validated with `sudo sshd -t` and `sudo sshd -T`, then restarted sshd.

### 5. Firewall and SELinux

~~~
sudo firewall-cmd --permanent --remove-service=cockpit
sudo firewall-cmd --reload
~~~

Allowed services: `ssh`, `dhcpv6-client`. SELinux: Enforcing.

### 6. Storage (LVM)

~~~
sudo pvcreate /dev/vdb
sudo vgcreate vg_data /dev/vdb
sudo lvcreate -n lv_data -L 1G vg_data
sudo mkfs.xfs /dev/vg_data/lv_data
sudo mkdir /data
echo '/dev/vg_data/lv_data /data xfs defaults 0 0' | sudo tee -a /etc/fstab
sudo findmnt --verify
sudo mount -a
sudo lvextend -r -L +500M /dev/vg_data/lv_data
~~~

Grown online from 1 GB to 1.5 GB; the mount survives a reboot.

## Verification

| Check | Result |
|---|---|
| `ssh rocky-01` | Logs in with key, no password |
| `ssh -o PubkeyAuthentication=no rocky-01` | Permission denied |
| `firewall-cmd --list-services` | dhcpv6-client ssh |
| `getenforce` | Enforcing |
| `df -h /data` after reboot | 1.5G mounted |

## Lessons learned

- Check the prompt before every command: `mac@Mac` is the Mac, `[yusuf@rocky-01]` is the VM.
- UTM cannot eject the ISO while the installer holds the drive; stop the VM first.
- Control + Option releases a mouse captured by a UTM window.
- The large linux-firmware package is not needed on a VM and can be excluded from updates.

## Next

Phase 1: rebuild everything above with Ansible roles.
