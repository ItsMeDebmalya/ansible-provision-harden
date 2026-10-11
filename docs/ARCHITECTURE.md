# Architecture

## Goal

Make server baseline-hardening **repeatable, auditable, and fast**: a single
playbook run brings any fresh Ubuntu or RHEL-family host to a known, secure,
production-ready state — with no per-host manual configuration.

## Execution flow

```
ansible-playbook site.yml
        │
        ▼
┌───────────────┐   ┌───────────┐   ┌──────────────────────────────┐
│   provision   │──▶│   patch   │──▶│            harden            │
│───────────────│   │───────────│   │──────────────────────────────│
│ timezone      │   │ apt/dnf   │   │ sshd drop-in (CIS 5.2.x)     │
│ chrony        │   │ upgrade   │   │ ufw / firewalld (CIS 3.5.x)  │
│ base packages │   │ autoremove│   │ sysctl network (CIS 3.x)     │
│ admin user    │   │ reboot?   │   │ login.defs ageing (CIS 5.4)  │
│ sudo + keys   │   │           │   │ auditd + fail2ban            │
└───────────────┘   └───────────┘   └──────────────────────────────┘
```

Each role is **independent and idempotent** and can be run on its own via tags
(`--tags provision|patch|harden`). Roles support both the Debian and RHEL
families by branching on `ansible_os_family`.

## Design decisions

- **Drop-in sshd config** (`/etc/ssh/sshd_config.d/99-hardening.conf`) instead
  of editing the main file — safer, and validated with `sshd -t` before it is
  written.
- **Default-deny firewall** with an explicit allowlist, so only intended ports
  are reachable.
- **Container-safe role flags** (`harden_in_container`, `provision_start_services`,
  `patch_upgrade`) let the exact same roles run under Molecule in CI without a
  real init system, while full behaviour runs on real hosts.
- **sudoers written with `validate: visudo -cf`** to guarantee a broken sudoers
  file can never be deployed.

## Testing strategy

| Layer | Tool | What it proves |
|-------|------|----------------|
| Static | ansible-lint | Style, deprecations, best practices |
| Functional | Molecule `converge` | Roles apply cleanly on Ubuntu + Rocky |
| Idempotence | Molecule idempotence | Second run reports **zero** changes |
| Assertion | Molecule `verify` | Hardened state is actually present |

All four run on every push and pull request via GitHub Actions.

## Extending

- Add a new control: create a task file under `roles/harden/tasks/`, include it
  from `main.yml`, expose knobs in `defaults/main.yml`, and assert it in
  `molecule/default/verify.yml`.
- Add a distro: add a platform to `molecule/default/molecule.yml` and branch
  any family-specific tasks on `ansible_os_family`.
```
