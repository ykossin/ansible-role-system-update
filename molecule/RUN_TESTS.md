# Запуск тестов Molecule

## Быстрый запуск

```bash
cd /home/kossin/OVR/jobs/kch/bindki-io/exchanger-iac/ansible/roles/system-update

# Запустить все тесты
MOLECULE_ROLE_NAME_CHECK=0 molecule test

# Или пошагово
MOLECULE_ROLE_NAME_CHECK=0 molecule create
MOLECULE_ROLE_NAME_CHECK=0 molecule converge
MOLECULE_ROLE_NAME_CHECK=0 molecule verify
MOLECULE_ROLE_NAME_CHECK=0 molecule destroy
```

## Используемые образы

Тесты используют образы из Docker Hub:
- `ykossin/docker-systemd:debian-bookworm`
- `ykossin/docker-systemd:debian-bullseye`
- `ykossin/docker-systemd:ubuntu-22.04`
- `ykossin/docker-systemd:ubuntu-20.04`

## Переменная окружения

`MOLECULE_ROLE_NAME_CHECK=0` - отключает проверку имени роли, которая не критична для локального тестирования.

## Проверка статуса тестов

```bash
# Проверить логи
tail -f /tmp/molecule_test.log

# Проверить статус контейнеров
podman ps | grep molecule

# Войти в контейнер для отладки
molecule login -h instance-debian-bookworm
```
