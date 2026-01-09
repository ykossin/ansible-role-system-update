# Быстрый старт - Molecule тесты

## Подготовка

### 1. Установка Molecule

```bash
pip install molecule molecule-plugins[docker]
```

### 2. Сборка образов docker-systemd

```bash
cd /home/kossin/OVR/jobs/me/docker-systemd

# Собрать все необходимые образы
make build DISTR=debian VERSION=bookworm
make build DISTR=debian VERSION=bullseye
make build DISTR=ubuntu VERSION=22.04
make build DISTR=ubuntu VERSION=20.04

# Проверить наличие образов
docker images | grep docker-systemd
```

### 3. Переход в директорию роли

```bash
cd /home/kossin/OVR/jobs/kch/bindki-io/exchanger-iac/ansible/roles/system-update
```

## Запуск тестов

### Полный цикл тестирования

```bash
molecule test
```

Эта команда выполнит:
1. `create` - создание контейнеров
2. `prepare` - подготовка окружения
3. `converge` - применение роли
4. `verify` - проверка результатов
5. `destroy` - удаление контейнеров

### Пошаговое выполнение

```bash
# 1. Создать контейнеры
molecule create

# 2. Подготовить окружение
molecule prepare

# 3. Применить роль
molecule converge

# 4. Запустить тесты
molecule verify

# 5. Удалить контейнеры
molecule destroy
```

### Тестирование на одной платформе

```bash
# Только Debian 12
molecule converge -- --limit instance-debian-bookworm
molecule verify -- --limit instance-debian-bookworm

# Только Ubuntu 22.04
molecule converge -- --limit instance-ubuntu-22.04
molecule verify -- --limit instance-ubuntu-22.04
```

## Отладка

### Вход в контейнер

```bash
# Сначала применить роль
molecule converge

# Войти в контейнер
molecule login -h instance-debian-bookworm

# Внутри контейнера можно проверить:
# - systemctl status
# - apt list --upgradable
# - cat /var/log/ansible/system-update-*.log
```

### Просмотр логов

```bash
# Логи Molecule
molecule --debug test

# Логи Docker контейнеров
docker logs molecule-system-update-instance-debian-bookworm
```

## Полезные команды

```bash
# Список всех команд
molecule --help

# Список сценариев
molecule list

# Проверка синтаксиса
molecule syntax

# Линтинг
molecule lint
```

## Устранение проблем

### Образы не найдены

```bash
# Проверить наличие образов
docker images | grep docker-systemd

# Если образов нет, собрать их
cd /home/kossin/OVR/jobs/me/docker-systemd
make build DISTR=debian VERSION=bookworm
```

### Ошибка доступа к Docker

```bash
# Добавить пользователя в группу docker
sudo usermod -aG docker $USER
newgrp docker
```

### Systemd не запускается

Проверьте, что в `molecule.yml` указаны правильные параметры:
- `privileged: true`
- `cgroupns_mode: host`
- Монтирование `/sys/fs/cgroup`

## Примеры использования

### Тестирование только обновления безопасности

Отредактируйте `molecule/default/converge.yml`:

```yaml
vars:
  system_security_only: true
  system_upgrade_packages: false
```

Запустите:
```bash
molecule converge
molecule verify
```

### Тестирование с автоматическими обновлениями

Отредактируйте `molecule/default/converge.yml`:

```yaml
vars:
  system_unattended_upgrades: true
  system_unattended_upgrades_config:
    mail_to: "test@example.com"
```

Запустите:
```bash
molecule converge
molecule verify
```

## Дополнительная информация

См. полную документацию в [README.md](README.md)
