# Astra-Стипендия 2025-2026: CryptoPro CSP для Astra Automation

Ansible Collection для базовой установки CryptoPro CSP из офлайн TGZ (DEB-пакеты) и базовой подготовки токен/смарт-карт окружения на Astra Linux.

## Роли

- `placeholder_namespace.cryptopro_csp_astra.cryptopro_csp` — установка CSP из TGZ + базовая проверка.
- `placeholder_namespace.cryptopro_csp_astra.cryptopro_csp_token` — установка PC/SC зависимостей и базовая проверка.

## Переменные

### Общие

- `cryptopro_csp_bin_dir`: `/opt/cprocsp/bin/amd64`
- `cryptopro_csp_sbin_dir`: `/opt/cprocsp/sbin/amd64`
- `cryptopro_csp_manage_path`: `false`

### Роль `cryptopro_csp`

- `cryptopro_csp_tgz_path`: `""` (обязательная при установке)
- `cryptopro_csp_tgz_on_remote`: `false`
- `cryptopro_csp_extract_dir`: `/tmp/cryptopro_csp_extract`
- `cryptopro_csp_install_dir`: `/tmp/cryptopro_csp_install` (deprecated, backward compatibility)
- `cryptopro_csp_force_reinstall`: `false`
- `cryptopro_csp_install_check_path`: `/opt/cprocsp/bin/amd64/cryptcp`
- `cryptopro_csp_cleanup`: `true`
- `cryptopro_csp_run_smoke_test`: `true`
- `cryptopro_csp_license_apply`: `false`
- `cryptopro_csp_license_key`: `""`

### Роль `cryptopro_csp_token`

- `cryptopro_csp_token_enable`: `true`
- `cryptopro_csp_token_packages`: `["pcscd", "libccid", "libgost-astra"]`
- `cryptopro_csp_pcscd_service`: `pcscd`
- `cryptopro_csp_token_run_smoke_test`: `true`

## Установка `cryptopro_csp` из TGZ

### 1) TGZ на controller (`cryptopro_csp_tgz_on_remote: false`)

```yaml
- name: Install CryptoPro CSP from controller TGZ
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: placeholder_namespace.cryptopro_csp_astra.cryptopro_csp
      vars:
        cryptopro_csp_tgz_path: /mnt/data/linux-amd64_deb.tgz
        cryptopro_csp_tgz_on_remote: false
        cryptopro_csp_extract_dir: /tmp/cryptopro_csp_extract
```

### 2) TGZ уже на target (`cryptopro_csp_tgz_on_remote: true`)

```yaml
- name: Install CryptoPro CSP from TGZ already on target
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: placeholder_namespace.cryptopro_csp_astra.cryptopro_csp
      vars:
        cryptopro_csp_tgz_path: /root/linux-amd64_deb.tgz
        cryptopro_csp_tgz_on_remote: true
        cryptopro_csp_extract_dir: /tmp/cryptopro_csp_extract
```

## Примеры playbooks

- `examples/minimal_install.yml`
- `examples/install_with_tokens.yml`
- `examples/install_with_license.yml`

Все examples используют FQCN ролей коллекции.

## Запуск examples

```bash
ansible-playbook -i <inventory> ansible_collections/placeholder_namespace/cryptopro_csp_astra/examples/minimal_install.yml
ansible-playbook -i <inventory> ansible_collections/placeholder_namespace/cryptopro_csp_astra/examples/install_with_tokens.yml
ansible-playbook -i <inventory> ansible_collections/placeholder_namespace/cryptopro_csp_astra/examples/install_with_license.yml
```

## Безопасность

- Лицензионный ключ передавайте через Ansible Vault.
- Задачи с ключом должны выполняться с `no_log: true`.

## Качество

- `make lint` — запуск `ansible-lint`
- `make syntax` — `ansible-playbook --syntax-check` для `examples/*.yml`
