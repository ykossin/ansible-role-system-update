# Ansible Role: System Update

Роль для обновления операционной системы Debian/Ubuntu.

## Описание

Эта роль выполняет обновление операционной системы:
- Обновление списка пакетов (apt update)
- Обновление установленных пакетов (apt upgrade)
- Опциональное обновление дистрибутива (dist-upgrade)
- Автоматическое удаление неиспользуемых пакетов
- Очистка кэша apt
- Настройка автоматических обновлений (unattended-upgrades)
- Перезагрузка после обновления ядра (опционально)

## Требования

- Debian 11+ или Ubuntu 20.04+
- Доступ root/sudo
- Ansible >= 2.9

## Переменные роли

См. `defaults/main.yml` для всех доступных переменных.

### Основные переменные

```yaml
# Включить обновление системы
system_update_enabled: true

# Обновлять пакеты
system_upgrade_packages: true

# Обновлять дистрибутив (dist-upgrade)
system_upgrade_distribution: false

# Удалять неиспользуемые пакеты
system_autoremove: true

# Перезагружать после обновления ядра
system_reboot_after_kernel_update: false

# Автоматические обновления (unattended-upgrades)
# ВАЖНО: По умолчанию ВЫКЛЮЧЕНО для безопасности
# Включите только если уверены, что автоматические обновления не нарушат работу сервисов
system_unattended_upgrades: false
```

## Использование

### Базовое обновление

```yaml
- hosts: all
  become: true
  roles:
    - system-update
```

### С перезагрузкой после обновления ядра

```yaml
- hosts: all
  become: true
  vars:
    system_reboot_after_kernel_update: true
  roles:
    - system-update
```

### Только безопасные обновления

```yaml
- hosts: all
  become: true
  vars:
    system_security_only: true
  roles:
    - system-update
```

### С автоматическими обновлениями (ОПЦИОНАЛЬНО)

**⚠️ ВНИМАНИЕ:** Автоматические обновления по умолчанию ВЫКЛЮЧЕНЫ.  
Включайте только если уверены, что автоматические обновления безопасны для ваших сервисов.

```yaml
- hosts: all
  become: true
  vars:
    # Явно включаем автоматические обновления
    system_unattended_upgrades: true
    system_unattended_upgrades_config:
      mail_to: "admin@example.com"
      mail_report: "on-change"
      mail_only_on_error: "false"
  roles:
    - system-update
```

## Теги

- `update` - все задачи обновления
- `system` - системные задачи
- `check` - проверка обновлений
- `upgrade` - обновление пакетов
- `dist-upgrade` - обновление дистрибутива
- `security` - безопасные обновления
- `autoremove` - удаление неиспользуемых пакетов
- `clean` - очистка кэша
- `reboot` - перезагрузка
- `unattended` - автоматические обновления

## Примеры использования тегов

```bash
# Только проверка обновлений
ansible-playbook playbook.yml --tags check

# Обновление без перезагрузки
ansible-playbook playbook.yml --tags upgrade --skip-tags reboot

# Только очистка кэша
ansible-playbook playbook.yml --tags clean
```

## Лицензия

ISC License

---

**Последнее обновление:** январь 2026
