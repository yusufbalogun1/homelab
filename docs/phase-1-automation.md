# Phase 1: Automation

Status: complete (12 Nov 2026)

## Goal

Rebuild the Phase 0 setup as Ansible code, so a fresh Rocky Linux VM is fully configured by one playbook run, and running it again changes nothing.

## Layout

~~~
ansible/
  ansible.cfg
  inventories/lab/hosts.yml
  inventories/lab/group_vars/lab/vars.yml
  inventories/lab/group_vars/lab/vault.yml   (encrypted)
  playbooks/site.yml
  roles/baseline    hostname, timezone, packages, chrony
  roles/hardening   SSH drop-in, firewalld, SELinux
  roles/storage     LVM volume, XFS filesystem, /data mount
~~~

## Roles

| Role | What it does |
|---|---|
| baseline | Sets hostname and timezone, installs core packages (including python3-libselinux), enables chronyd |
| hardening | Key-only SSH with root login off, firewall allows SSH and blocks Cockpit, SELinux enforcing |
| storage | Creates vg_data and lv_data on /dev/vdb, formats XFS, mounts /data and persists it in fstab |

## Running it

~~~
cd ansible
ansible-playbook playbooks/site.yml                   # all lab hosts
ansible-playbook playbooks/site.yml --limit rocky-02  # one host
ansible-playbook playbooks/site.yml --check --diff    # dry run
~~~

The sudo password is stored with Ansible Vault; the vault password file lives outside the repo.

## Verification

| Test | Result |
|---|---|
| Fresh VM (rocky-02), first run | ok=16 changed=8 failed=0 |
| Same VM, second run | changed=0 |
| Existing VM (rocky-01), run against manual setup | changed=0 |
| `df -h /data` on rocky-02 | 1.5G mounted |
| Password SSH login on rocky-02 | Permission denied |

## Lessons learned

- Rocky Minimal lacks python3-libselinux, which Ansible's SELinux module needs; the baseline role now installs it.
- When a play fails, pending handlers are dropped; `--force-handlers` runs them anyway.
- Check mode skips `command` tasks because Ansible cannot predict their effect.
- `UNREACHABLE` after an ad-hoc `poweroff` is expected: the host shuts down mid-connection.
- Running roles against the hand-built VM first proved the code matched reality before using it on a clean machine.

## Next

Phase 2: a k3s cluster on Oracle Cloud Always Free Arm VMs, built by a new k3s role.
