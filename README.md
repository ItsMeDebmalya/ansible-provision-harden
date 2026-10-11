# ansible-provision-harden

[![CI](https://github.com/ItsMeDebmalya/ansible-provision-harden/actions/workflows/ci.yml/badge.svg)](https://github.com/ItsMeDebmalya/ansible-provision-harden/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> Bring a fresh Linux host to a **security-baselined, production-ready** state in minutes — provisioned, patched, and CIS-hardened from a single playbook run.

One `ansible-playbook site.yml` run takes a clean Ubuntu or RHEL-family server
from first boot to a hardened baseline: base packages and an admin user,
all available updates, and a CIS-aligned security configuration (SSH, host
firewall, kernel/sysctl, password policy, auditing, and intrusion prevention).
Everything is **idempotent** — re-running changes nothing — and verified in CI
with **Molecule** across Ubuntu 22.04 and Rocky Linux 9.

---

## What it does

| Role | Responsibility | Key CIS areas |
|------|----------------|---------------|
| **provision** | Timezone, time sync (chrony), base packages, admin user + passwordless sudo, authorised keys | baseline |
| **patch** | Full package/security upgrade (apt/dnf), autoremove, optional reboot-if-required | patching |
| **harden** | SSH hardening, host firewall (ufw/firewalld), sysctl network hardening, password ageing policy, auditd, fail2ban | 3.x, 5.2.x, 5.4.x |

## Repository layout

```
.
├── site.yml                 # master playbook (provision → patch → harden)
├── ansible.cfg
├── requirements.yml         # Galaxy collections (ansible.posix, community.general)
├── inventory/hosts.ini
├── group_vars/all.yml       # tunable knobs for every host
├── roles/
│   ├── provision/
│   ├── patch/
│   └── harden/              # templates/, tasks/, defaults/, handlers/
├── molecule/default/        # molecule.yml, converge.yml, verify.yml
└── .github/workflows/ci.yml # ansible-lint + molecule on every push/PR
```

## Quick start

```bash
# 1. Install dependencies
pip install ansible-core
ansible-galaxy collection install -r requirements.yml

# 2. Point inventory/hosts.ini at your server(s) and set your SSH key in
#    group_vars/all.yml (provision_admin_authorized_keys).

# 3. Bring every host to the hardened baseline
ansible-playbook site.yml

# Run a single stage
ansible-playbook site.yml --tags harden
```

## Configuration

All behaviour is tunable via variables in `group_vars/all.yml` (and each
role's `defaults/main.yml`). Highlights:

| Variable | Default | Purpose |
|----------|---------|---------|
| `harden_ssh_permit_root_login` | `no` | Disable direct root SSH |
| `harden_ssh_password_authentication` | `no` | Key-only authentication |
| `harden_ssh_allow_groups` | `ssh-users` | Restrict SSH to a group |
| `harden_firewall_allowed_tcp_ports` | `[22]` | Default-deny inbound, allowlist |
| `harden_password_max_days` | `365` | Password ageing (login.defs) |
| `harden_enable_auditd` / `harden_enable_fail2ban` | `true` | Auditing / brute-force protection |
| `patch_reboot_if_required` | `false` | Reboot after patching if needed |

## Testing

Roles are tested with **Molecule** (Docker driver) against Ubuntu 22.04 and
Rocky Linux 9. The sequence runs `converge` (apply), an **idempotence** check
(second run must report zero changes), and a `verify` play that asserts the
hardened state (sshd directives, password policy, admin user, auditd).

```bash
pip install "molecule-plugins[docker]" molecule ansible-core
molecule test      # full create → converge → idempotence → verify → destroy
molecule converge  # apply and keep the containers for inspection
```

> Container-unsafe steps (service enablement, firewall, sysctl reload) are
> skipped inside test containers via role flags and exercised on real hosts
> through `site.yml`.

## Verifying hardening on a real host

Run a Lynis audit before and after to see the baseline improvement:

```bash
sudo apt install -y lynis    # or: sudo dnf install -y lynis
sudo lynis audit system      # note the "Hardening index" before and after
```

## License

MIT — see [LICENSE](LICENSE).
