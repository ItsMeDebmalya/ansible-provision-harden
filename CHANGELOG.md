# Changelog

All notable changes to this project are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Added
- Initial project scaffolding: `ansible.cfg`, inventory, `group_vars`, `site.yml`.
- `provision` role: timezone, chrony time sync, base packages, admin user with
  passwordless sudo and authorised keys.
- `patch` role: cross-distro package/security upgrade with optional reboot.
- `harden` role: CIS-aligned SSH hardening, host firewall (ufw/firewalld),
  sysctl network hardening, password ageing policy, auditd, and fail2ban.
- Molecule test scenario (Ubuntu 22.04 + Rocky Linux 9) with converge,
  idempotence, and verify stages.
- GitHub Actions CI running ansible-lint and Molecule on every push and PR.
- Project documentation: README and `docs/ARCHITECTURE.md`.
