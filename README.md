# Homelab

A small, fully automated platform lab built step by step and rebuildable from code: Linux, Ansible, k3s, an API gateway and observability.

## Architecture

- **Control machine:** MacBook Pro (Apple M2) running Ansible
- **rocky-01:** Rocky Linux 9 (aarch64) VM in UTM for Linux and Ansible practice
- **Cluster (Phase 2+):** k3s on Oracle Cloud Always Free Arm VMs
- **Cloud extension (Phase 5):** Azure

## Phases

| Phase | Focus | Status |
|---|---|---|
| 0 | Foundations: VM, SSH, networking, firewall, storage | Done |
| 1 | Ansible automation | Next |
| 2 | k3s cluster | Planned |
| 3 | Demo services behind Kong | Planned |
| 4 | Prometheus and Grafana | Planned |
| 5 | Azure extension | Planned |

## Docs

- [Phase 0: Foundations](docs/phase-0-foundations.md)
