[![Molecule](https://github.com/iamenr0s/ansible-role-file-integrity/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-file-integrity/actions/workflows/molecule.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/file_integrity) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-file-integrity/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-file-integrity)

# Ansible Role: File Integrity

This role sets up audit-first AIDE file integrity monitoring on Debian/Ubuntu, RHEL/AlmaLinux/RockyLinux and Fedora servers. It installs AIDE, schedules `aide --check` with a systemd timer, and (only with your consent) builds the baseline database. It never accepts changes into the baseline on its own.

Features
--------
- Audit mode by default: read-only, and reports every deviation. Changes need `linux_hardening_confirm_remediation: true`.
- Installs `aide` (plus `aide-common` on Debian/Ubuntu) and writes `linux-hardening-aide-check.service` / `.timer`.
- Builds the baseline only with `linux_hardening_fi_init_database: true`, and never replaces an existing one.
- A failed scheduled check fails the unit, so it shows in `systemctl --failed` and the journal.
- The only role in the `iamenr0s.linux_hardening` collection that manages AIDE.

## What it checks

| Item | Remediation |
|---|---|
| `aide` installed (`aide-common` too on Debian) | installed from the distribution repositories |
| Baseline database (`/var/lib/aide/aide.db[.gz]`) | **needs `linux_hardening_fi_init_database: true`**: `aide --init`, then the new database becomes the baseline. Otherwise **reported only** |
| `linux-hardening-aide-check.service` / `.timer` in `/etc/systemd/system` | written (`OnCalendar` = `linux_hardening_fi_schedule`, default `daily`, random delay up to 1 h, `Persistent=true`) |
| Timer enabled and active | enabled and started |
| With `linux_hardening_fi_run_check: true`: machine matches the baseline | **reported only** (`aide --check` exit 1-7: files differ) |

The check unit runs `aide --check`; AIDE exits non-zero when files differ, so the unit **fails** and shows up in `systemctl --failed` and the journal. The role sends no mail and writes no alerting: wire that to the failed unit yourself.

## Variable names
Variables use the collection-wide `linux_hardening_*` prefix (shared with sibling roles) rather than `file_integrity_*`. Full list: `defaults/main.yml` and `meta/argument_specs.yml`.

## Modes
`audit` (default) is read-only and sets `linux_hardening_fi_findings` (`key, expected, actual, fixable`). `remediate` needs `linux_hardening_confirm_remediation: true`; afterwards only fixable findings fail the run.

## Safety
- The baseline records the machine **as it is now** and treats that as trusted. Building it on a machine you have not vetted bakes a compromise into the baseline, hence the consent switch. It is built only when no database exists; an existing baseline is never replaced, so the role cannot be used to wave changes through.
- Updating the baseline after legitimate changes (`aide --update`, then replacing the database) is a manual step by design.
- The baseline is built last, after the role's own units are in place, so they are part of it. The first scan reads every file in the AIDE configuration: minutes of I/O on a real machine. The scheduled check runs with `Nice=19` and idle I/O class.
- The AIDE configuration itself (`/etc/aide.conf`, `/etc/aide/aide.conf`) is **not** managed: what is watched is the distribution default. Paths are the packages' defaults; a changed `database_in` is not followed.

## Example
```yaml
- hosts: all
  become: true
  roles:
    - role: iamenr0s.file_integrity
      vars:
        linux_hardening_mode: remediate
        linux_hardening_confirm_remediation: true
        linux_hardening_fi_init_database: true
        linux_hardening_fi_schedule: "*-*-* 03:30:00"
```

## Verification
Container scenarios `default` (package, units and timer; no baseline, which must be reported as not fixable), `audit` (read-only), `refuse`. **VM scenario `vagrant`** (libvirt; `sg libvirt -c "molecule test -s vagrant"`; box `MOLECULE_VAGRANT_BOX`): builds a real baseline, checks that the machine matches it, edits `/etc/hosts`, checks that the audit reports the change as not fixable and that the scheduled unit fails, runs the role again (the change must **not** be absorbed into the baseline), reverts, **reboots** and checks the timer and baseline survived. CI matrix: AlmaLinux 8–10, Rocky 8–10, Fedora 42–44, Debian 12–13, Ubuntu 22.04/24.04.

## Rollback
`systemctl disable --now linux-hardening-aide-check.timer`, delete the two units, `systemctl daemon-reload`; remove the package and `/var/lib/aide` if wanted.

## Known gaps
- Debian and Ubuntu also ship their own `dailyaidecheck.timer`; it is left alone, so both run there.
- No report parsing, mail or log shipping; no AIDE configuration tuning (exclusions, rules); no baseline update workflow.
- A baseline taken while Ansible is running includes Ansible's own temporary files (and the distribution's default rules are noisy on `/tmp`, `/run`, logs); tune the AIDE configuration for your estate, the role does not.
- CIS IDs not mapped.

CI & Release (maintainers)
----------------------------
A single workflow (`.github/workflows/molecule.yml`) runs lint and the full Molecule distro matrix on pushes to `main`, PRs, and `v*` tags. On `v*` tags, a `release` job publishes to Ansible Galaxy after all tests pass.

One-time setup:

```bash
# Galaxy publishing key (environment-scoped, get it from galaxy.ansible.com/ui/token)
gh secret set GALAXY_API_KEY --repo iamenr0s/ansible-role-file-integrity

# Code scanning notifications (Slack webhook URL; for Discord append /slack to the webhook URL)
gh secret set SECURITY_ALERT_WEBHOOK --repo iamenr0s/ansible-role-file-integrity
```

`.github/workflows/code-scanning-notify.yml` polls the code-scanning API every 6 hours and posts new or updated open alerts to that webhook (GitHub Actions cannot trigger on `code_scanning_alert` directly).

To release: tag a commit `vX.Y.Z` and push the tag — CI gates the Galaxy publish.

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the
local pipeline commands and pull request checklist. This project follows the
[Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).

## Security

See [SECURITY.md](SECURITY.md) — GitHub private vulnerability reporting, no
public issues for security bugs.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

## License

This project is licensed under the [MIT License](LICENSE).

## Author Information

Author: iamenr0s

Galaxy: `iamenr0s.file_integrity`
