# Phase 2: Cluster

Status: complete (18 Nov 2026)

## Goal

A two-node k3s cluster on Oracle Cloud Always Free Arm VMs, built by Ansible, with the existing baseline and hardening roles applied to the cloud nodes.

## Environment

| Item | Value |
|---|---|
| Cloud | Oracle Cloud Infrastructure, UK South (London), Always Free |
| Network | VCN 10.0.0.0/16, public subnet 10.0.0.0/24 |
| Nodes | oci-01 (server), oci-02 (agent); VM.Standard.A1.Flex, 1 OCPU and 6 GB each |
| OS | Oracle Linux 9.8 (aarch64), user `opc` |
| Kubernetes | k3s v1.37.2, containerd 2.3.7 |

## Security

- SSH (port 22) allowed only from my current public IP (/32)
- All protocols allowed within 10.0.0.0/16 for node-to-node traffic
- On each node, firewalld trusts the node, pod (10.42.0.0/16) and service (10.43.0.0/16) networks
- Baseline and hardening roles applied: key-only SSH, root login off, SELinux enforcing

## Ansible changes

- Inventory groups: `oci`, `k3s_server`, `k3s_agents`
- `group_vars/oci/vars.yml` sets the timezone to UTC for cloud servers
- `site.yml` split into two plays: base configuration for all hosts, storage for lab VMs only
- New `k3s` role: firewalld trust, install script, server install, token handoff to the agent via `slurp` and `hostvars`

## Verification

| Test | Result |
|---|---|
| `site.yml --limit oci`, second run | changed=0 on both nodes |
| `k3s.yml`, second run | changed=0, no warnings |
| `kubectl get nodes` | oci-01 control-plane Ready, oci-02 Ready |
| 2-replica nginx deployment | one pod on each node, both Running |

## Lessons learned

- A home connection's public IP changes; when SSH to the cloud times out, check `curl ifconfig.me` and update the /32 rule first.
- Oracle sets the hostname from the VNIC name; the baseline role corrects it.
- `sudo` on RHEL-family systems excludes /usr/local/bin from its path, so call `/usr/local/bin/k3s` in full.
- `ansible_default_ipv4`-style fact variables are deprecated; use `ansible_facts['default_ipv4']`.
- The cluster token moves from server to agent at run time and is never stored in the repo.

## Next

Phase 3: two demo services behind a Kong API gateway with key auth and rate limiting.
