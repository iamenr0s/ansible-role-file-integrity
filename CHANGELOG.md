# Changelog

## Unreleased
- Fix `remediate` under `--check`: no longer fails on the unwritten timer; skips the re-audit and baseline build.
- Work with `gather_facts: false` (gathers the minimal subset itself).
- Write the units with the `template` module; CoP style fixes (sub-task name prefixes, wrapped long lines, block-style argument specs).
- Molecule `default` runs a `check` step before converge, and converges with `gather_facts: false`.
- An audit under `--check` still runs `aide --check` when `linux_hardening_fi_run_check` is set.
- Fix `remediate --check` on Debian/Ubuntu: the package index is refreshed even in check mode, so apt can preview the install.
- Audit builds its findings with one task per check instead of one Jinja block (same findings); unit drift now also covers owner and mode.
- Install AIDE with one `package` task on both families; split the audit failure into one task per mode.

## 1.0.0
- Split out of the iamenr0s.linux_hardening collection; no functional change.
