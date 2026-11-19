# Changelog

## Unreleased
- Fix `remediate` under `--check`: no longer fails on the unwritten timer; skips the re-audit and baseline build.
- Work with `gather_facts: false` (gathers the minimal subset itself).
- Write the units with the `template` module; CoP style fixes (sub-task name prefixes, wrapped long lines, block-style argument specs).
- Molecule `default` runs a `check` step before converge, and converges with `gather_facts: false`.
- An audit under `--check` still runs `aide --check` when `linux_hardening_fi_run_check` is set.

## 1.0.0
- Split out of the iamenr0s.linux_hardening collection; no functional change.
