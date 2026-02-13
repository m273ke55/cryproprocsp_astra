# cryptopro_csp role

При установке из TGZ роль ставит все DEB одним `apt-get install` в одной транзакции.
Если в архиве есть `lsb-cprocsp-ca-certs`, но нет `lsb-cprocsp-capilite-noarch`, пакет `ca-certs` исключается (по умолчанию `cryptopro_csp_skip_ca_certs_without_noarch: true`).
