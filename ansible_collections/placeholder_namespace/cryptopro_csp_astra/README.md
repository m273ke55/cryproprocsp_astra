# Astra-Стипендия 2025-2026: CryptoPro CSP для Astra Automation

Ansible-collection (MVP) для базовой установки и базовой проверки готовности CryptoPro CSP на Astra Linux.

## Состав коллекции

Ровно 2 роли:

- `cryptopro_csp` — установка CSP из офлайн TGZ-дистрибутива (`linux-amd64_deb.tgz`) и базовая проверка.
- `cryptopro_csp_token` — подготовка зависимостей токенов/смарт-карт (PC/SC) и базовая проверка.

> В этой итерации реализован **каркас**. Логика установки, применения лицензии и расширенной верификации оставлена как TODO для следующей итерации.

## Scope MVP

- Только установка из TGZ с DEB-пакетами.
- Поддержка типовых путей `/opt/cprocsp/...` и утилит `cpconfig`, `certmgr`, `csptest` в структуре роли.
- Без ZPS, `initramfs`, reboot, правок `config64.ini`, destructive-операций с контейнерами, автоустановки Rutoken/JaCarta драйверов.

## Быстрый старт

См. примеры в `examples/`:

- `minimal_install.yml`
- `install_with_tokens.yml`
- `install_with_license.yml`

## Переменные

### Общие

- `cryptopro_csp_bin_dir`: `/opt/cprocsp/bin/amd64`
- `cryptopro_csp_sbin_dir`: `/opt/cprocsp/sbin/amd64`
- `cryptopro_csp_manage_path`: `false`

### Роль `cryptopro_csp`

- `cryptopro_csp_tgz_path`: обязательная (путь к TGZ)
- `cryptopro_csp_tgz_on_remote`: `false`
- `cryptopro_csp_extract_dir`: `/tmp/cryptopro_csp_extract`
- `cryptopro_csp_install_dir`: `/tmp/cryptopro_csp_install` (deprecated, backward compatibility)
- `cryptopro_csp_force_reinstall`: `false`
- `cryptopro_csp_run_smoke_test`: `true`
- `cryptopro_csp_license_apply`: `false`
- `cryptopro_csp_license_key`: `""`

### Роль `cryptopro_csp_token`

- `cryptopro_csp_token_enable`: `true`
- `cryptopro_csp_token_packages`: `["pcscd", "libccid", "libgost-astra"]`
- `cryptopro_csp_pcscd_service`: `pcscd`
- `cryptopro_csp_token_run_smoke_test`: `true`

## Безопасность

- Лицензионный ключ должен передаваться через Ansible Vault.
- Задачи, где используется ключ лицензии, помечаются `no_log: true`.

## Проверка качества

- `make lint` — запуск `ansible-lint`
- `make syntax` — `ansible-playbook --syntax-check` для playbook'ов из `examples/`

## Установка `cryptopro_csp` из TGZ

### 1) TGZ на controller (`cryptopro_csp_tgz_on_remote: false`)

```yaml
- name: Install CryptoPro CSP from controller TGZ
  hosts: all
  become: true
  gather_facts: true
  roles:
    - role: cryptopro_csp
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
    - role: cryptopro_csp
      vars:
        cryptopro_csp_tgz_path: /root/linux-amd64_deb.tgz
        cryptopro_csp_tgz_on_remote: true
        cryptopro_csp_extract_dir: /tmp/cryptopro_csp_extract
```

Роль распаковывает архив, устанавливает `*.deb` пакеты, проверяет маркер `cryptopro_csp_install_check_path` и повторный запуск без `cryptopro_csp_force_reinstall` не выполняет переустановку.
