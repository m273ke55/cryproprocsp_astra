# AGENTS.md — CryptoPro CSP Ansible Collection (Astra Automation, Astra-Стипендия 2025-2026)

## Goal
Implement an Ansible collection that provides **basic installation and basic system readiness** for **CryptoPro CSP** on **Astra Linux**.

We intentionally keep the collection small: **only 2 roles**:
1) `cryptopro_csp` — installs CryptoPro CSP from an offline `.tgz` distribution and performs basic verification.
2) `cryptopro_csp_token` — installs and configures token/smart-card dependencies (PC/SC) and performs basic verification.

The collection must be suitable for use in CI and repeated runs (idempotent).

## Supported installation source
- **Only TGZ** distribution that contains DEB packages (e.g. `linux-amd64_deb.tgz`).
- The TGZ file is provided by the user (downloaded manually from vendor site) and is accessible on the target host or copied by Ansible.

No online repository installation. No GUI installer.

## Operating System / Platform assumptions
- Astra Linux (amd64). Use `ansible_facts` to validate architecture.
- Use `become: true` when required.
- Prefer standard Ansible modules over raw shell. Use `command` only when needed.
- Avoid interactive prompts. All actions must be non-interactive.

## Scope (MVP)
### Role: cryptopro_csp (install + basic readiness)
Must:
- Accept a path to TGZ and install CryptoPro CSP non-interactively.
- Be idempotent by default (second run does not reinstall).
- Provide `cryptopro_csp_force_reinstall` to reinstall when explicitly requested.
- Provide a `cryptopro_csp_manage_path` option:
  - If true: create `/etc/profile.d/cryptopro-csp.sh` (or equivalent) to add `/opt/cprocsp/.../bin/amd64` and `/opt/cprocsp/.../sbin/amd64` to PATH.
  - If false: tasks still must work using absolute paths.
- Support license management as **optional**:
  - Can show license status.
  - Can set license key if provided.
  - Must never print license key to logs (`no_log: true` on tasks that contain key).
- Provide basic verification (smoke-test):
  - Check existence of expected binaries (cpconfig, certmgr, csptest).
  - Run at least one non-destructive command to confirm CSP is functional (e.g. `cpconfig -license -view` or `cpconfig -hardware reader -view`, whichever is safer to run without hardware).
  - Verification can be disabled via variable.

Must NOT:
- Do ZPS (Astra SE legacy key), update-initramfs, reboot.
- Do GOST-2001 warnings tweaks (config64.ini edits).
- Do container destructive operations (delete, change PIN, key copy).
- Do certificate import by default (can be out-of-scope for MVP).

### Role: cryptopro_csp_token (PC/SC deps)
Must:
- Install required OS packages for smart-card/token support:
  - `pcscd`, `libccid`, `libgost-astra` (configurable list).
- Ensure `pcscd` service is enabled and running (systemd).
- Provide verification:
  - Check `pcscd` is active.
  - Optionally run non-destructive reader enumeration via CryptoPro tools if available.
- Be safe if role runs before CryptoPro is installed (it should not fail just because CSP tools are missing).

Must NOT:
- Install vendor-specific token drivers (Rutoken/JaCarta) unless user provides packages explicitly (out-of-scope for MVP).

## Variables (names must match exactly)
### Common
- `cryptopro_csp_bin_dir`: default `/opt/cprocsp/bin/amd64`
- `cryptopro_csp_sbin_dir`: default `/opt/cprocsp/sbin/amd64`
- `cryptopro_csp_manage_path`: default `false` (prefer absolute paths by default)

### cryptopro_csp role
- `cryptopro_csp_tgz_path`: (required) path to tgz on controller or already on host
- `cryptopro_csp_tgz_on_remote`: default `false`
  - if false: copy TGZ from controller to remote temp dir
  - if true: TGZ path is already on remote
- `cryptopro_csp_install_dir`: default `/tmp/cryptopro_csp_install` (remote temp extraction dir)
- `cryptopro_csp_force_reinstall`: default `false`
- `cryptopro_csp_run_smoke_test`: default `true`

License:
- `cryptopro_csp_license_apply`: default `false`
- `cryptopro_csp_license_key`: default empty string
- Always `no_log: true` when key is used.

### cryptopro_csp_token role
- `cryptopro_csp_token_enable`: default `true`
- `cryptopro_csp_token_packages`: default `["pcscd","libccid","libgost-astra"]`
- `cryptopro_csp_pcscd_service`: default `pcscd`
- `cryptopro_csp_token_run_smoke_test`: default `true`

## Idempotency rules
- The installer must not run again if CSP is already installed and `cryptopro_csp_force_reinstall` is false.
- Determine "already installed" by at least one robust check, e.g.:
  - presence of `/opt/cprocsp/sbin/amd64/cpconfig` OR
  - `dpkg -s` on a known package if stable in your environment.
- Report `changed: false` on subsequent runs.

## Implementation guidance
- Use `unarchive` for tgz extraction to remote temp dir.
- Detect the correct installer script path after extraction and run it with `command`.
- Handle errors with clear messages.
- Keep tasks split into files:
  - `tasks/install.yml`, `tasks/path.yml`, `tasks/license.yml`, `tasks/verify.yml` for `cryptopro_csp`
  - `tasks/main.yml`, `tasks/verify.yml` for `cryptopro_csp_token`
- Provide `handlers` only if needed (e.g. restart pcscd).
- Use `become: true` for package install and system file changes.

## Documentation
Provide:
- `README.md` with usage examples for both roles, required vars, and security notes.
- `examples/` playbooks:
  - `minimal_install.yml`
  - `install_with_tokens.yml`
  - `install_with_license.yml` (license key via vault)
- Keep docs concise and focused on MVP.

## Testing / Quality
- Add `ansible-lint` configuration and ensure collection passes basic linting.
- Add Molecule scenario(s) if feasible; if not feasible on Astra, at least include a `make lint` and a simple syntax-check target.

## Acceptance Criteria (Definition of Done)
1) Running `cryptopro_csp` with a valid TGZ installs CSP and smoke-test passes.
2) Re-running the same play results in **no reinstallation** and `changed=0` for install tasks (unless `cryptopro_csp_force_reinstall=true`).
3) Running `cryptopro_csp_token` installs PC/SC deps and ensures `pcscd` active.
4) License apply, if enabled, does not leak key in logs and is idempotent (does not reapply if already set, as much as possible).
5) Examples in `examples/` work with documented variables.

## Out of scope
- ZPS mode / legacy keys / initramfs updates / reboot.
- GOST-2001 warnings suppression tweaks.
- Advanced container operations (delete, key copy, PIN change).
- Vendor token drivers auto-install (Rutoken/JaCarta), unless provided by user (not MVP).
- Full certificate lifecycle management (optional future phase).

