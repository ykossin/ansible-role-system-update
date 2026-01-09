# Molecule тесты для роли system-update

## Описание

Molecule тесты для проверки работы роли `system-update` на различных дистрибутивах Debian и Ubuntu.

## Требования

1. **Molecule** - установлен и настроен
   ```bash
   pip install molecule molecule-plugins[docker]
   ```

2. **Docker** - для запуска контейнеров
   ```bash
   # Проверка установки
   docker --version
   ```

3. **Образы docker-systemd** - должны быть собраны локально
   ```bash
   # Перейти в директорию проекта docker-systemd
   cd /home/kossin/OVR/jobs/me/docker-systemd
   
   # Собрать образы для тестирования
   make build DISTR=debian VERSION=bookworm
   make build DISTR=debian VERSION=bullseye
   make build DISTR=ubuntu VERSION=22.04
   make build DISTR=ubuntu VERSION=20.04
   ```

## Использование

### Настройка dependency (источники ролей)

Molecule поддерживает несколько источников ролей:

1. **Локальный каталог** - роли из файловой системы
2. **GitLab репозиторий** - роли из GitLab (публичные или приватные)
3. **Ansible Galaxy** - роли из официального репозитория

**Файл:** `molecule/default/requirements.yml`

**Пример для локальных ролей:**
```yaml
---
- src: ../../firewall
  name: firewall
  scm: local

- src: ../../fail2ban
  name: fail2ban
  scm: local
```

**Пример для GitLab:**
```yaml
---
- src: git+https://gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: main
```

**Подробнее:** См. [DEPENDENCY_GUIDE.md](DEPENDENCY_GUIDE.md) и [DEPENDENCY_EXAMPLES.md](DEPENDENCY_EXAMPLES.md)

### Запуск всех тестов

```bash
cd /home/kossin/OVR/jobs/kch/bindki-io/exchanger-iac/ansible/roles/system-update
molecule test
```

### Отдельные команды

```bash
# Создание контейнеров
molecule create

# Применение роли
molecule converge

# Запуск тестов
molecule verify

# Удаление контейнеров
molecule destroy

# Запуск только на одном инстансе
molecule converge -- --limit instance-debian-bookworm
molecule verify -- --limit instance-debian-bookworm
```

### Интерактивная отладка

```bash
# Создать контейнеры и применить роль
molecule converge

# Войти в контейнер для отладки
molecule login -h instance-debian-bookworm

# Запустить тесты
molecule verify

# Удалить контейнеры
molecule destroy
```

## Платформы для тестирования

Тесты запускаются на следующих платформах:

- **Debian 12 (bookworm)** - `instance-debian-bookworm`
- **Debian 11 (bullseye)** - `instance-debian-bullseye`
- **Ubuntu 22.04** - `instance-ubuntu-22.04`
- **Ubuntu 20.04** - `instance-ubuntu-20.04`

## Что тестируется

### Подготовка (prepare.yml)
- Запуск systemd
- Проверка свободного места
- Проверка версии дистрибутива
- Обновление списка пакетов

### Применение (converge.yml)
- Применение роли system-update
- Проверка создания логов
- Проверка статуса apt

### Верификация (verify.yml)
- Проверка версии дистрибутива
- Проверка наличия Python3 и apt
- Проверка свободного места
- Проверка целостности пакетов
- Проверка зависимостей apt
- Проверка создания директории для логов
- Проверка наличия логов обновления
- Проверка доступности репозиториев
- Проверка работы systemd

## Настройка

### Изменение платформ

Отредактируйте `molecule/default/molecule.yml` для добавления или удаления платформ.

### Изменение переменных роли

Отредактируйте `molecule/default/converge.yml` для изменения переменных роли при тестировании.

### Добавление тестов

Добавьте новые тесты в `molecule/default/verify.yml`.

## Устранение проблем

### Ошибка: "Image not found"

Убедитесь, что образы docker-systemd собраны:
```bash
docker images | grep docker-systemd
```

Если образов нет, соберите их:
```bash
cd /home/kossin/OVR/jobs/me/docker-systemd
make build DISTR=debian VERSION=bookworm
```

### Ошибка: "Systemd not running"

Проверьте, что контейнер запущен с правильными параметрами:
- `privileged: true`
- `cgroupns_mode: host`
- Монтирование `/sys/fs/cgroup`

### Ошибка: "Connection timeout"

Увеличьте таймаут в `molecule.yml`:
```yaml
provisioner:
  config_options:
    defaults:
      timeout: 60
```

## Интеграция с CI/CD

Пример для GitLab CI:

```yaml
test:molecule:
  image: quay.io/ansible/molecule:latest
  script:
    - cd roles/system-update
    - molecule test
  only:
    - merge_requests
    - main
```

## Дополнительная информация

- [Molecule документация](https://molecule.readthedocs.io/)
- [Molecule Docker driver](https://molecule.readthedocs.io/en/latest/drivers/docker.html)
- [Проект docker-systemd](../../../../me/docker-systemd/README.md)
