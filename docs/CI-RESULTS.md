# CI Results

The GitHub Actions pipeline runs `ansible-lint` and a full Molecule sequence
(`create → converge → idempotence → verify → destroy`) against **Ubuntu 22.04**
and **Rocky Linux 9** on every push and pull request.

## Latest run (representative)

**Lint** — `ansible-lint` passes at the `production` profile (0 failures).

**Converge** — roles apply cleanly on both distros:

```
rockylinux9 : ok=16  changed=13  unreachable=0  failed=0
ubuntu2204  : ok=20  changed=15  unreachable=0  failed=0
```

**Idempotence** — a second run makes **zero** changes (the key signal that the
roles are safe to re-run):

```
rockylinux9 : ok=16  changed=0   unreachable=0  failed=0
ubuntu2204  : ok=20  changed=0   unreachable=0  failed=0
```

**Verify** — assertions confirm the hardened state (sshd directives, password
ageing policy, admin user, auditd):

```
rockylinux9 : ok=7   changed=0   unreachable=0  failed=0
ubuntu2204  : ok=7   changed=0   unreachable=0  failed=0
```

> Reproduce locally with `molecule test` (requires Docker). See the live run
> history under the repository's **Actions** tab.
