# system-update

Роль для обновления операционной системы на серверах Debian и Ubuntu. Делает всё необходимое: проверяет систему перед обновлением, обновляет пакеты, проверяет что ничего не сломалось, и может настроить автоматические обновления.

## Что делает эта роль

Роль выполняет обновление системы в несколько этапов:

1. **Проверки перед обновлением** - проверяет версию дистрибутива, свободное место на диске, доступность репозиториев
2. **Обновление пакетов** - обновляет все пакеты или только указанные, может делать обычное обновление или dist-upgrade
3. **Проверка целостности** - после обновления проверяет что все пакеты в порядке и зависимости не сломались
4. **Логирование** - сохраняет информацию о том, что было обновлено
5. **Перезагрузка** - если обновилось ядро, может перезагрузить сервер (опционально)
6. **Автоматические обновления** - может настроить unattended-upgrades для автоматического обновления в будущем

## Требования

- Ansible 2.9 или новее
- Debian 11+ (Bullseye, Bookworm, Trixie) или Ubuntu 20.04+ (Focal, Jammy, Noble)
- Права sudo на целевом хосте

## Базовое использование

Самый простой способ - просто добавить роль в playbook:

```yaml
- hosts: servers
  become: yes
  roles:
    - system-update
```

Это обновит все пакеты на сервере, но не будет перезагружать и не настроит автоматические обновления.

## Основные параметры

### Включение/выключение обновления

```yaml
system_update_enabled: true  # включить обновление (по умолчанию true)
```

Если поставить `false`, роль ничего не будет делать. Полезно когда нужно временно отключить обновления в playbook.

### Тип обновления

```yaml
system_upgrade_packages: true        # обновлять пакеты (по умолчанию true)
system_upgrade_distribution: false  # делать dist-upgrade вместо обычного upgrade (по умолчанию false)
system_security_only: false          # обновлять только security updates (по умолчанию false)
```

Обычно используется `system_upgrade_packages: true` с `system_upgrade_distribution: false` - это стандартное `apt upgrade`. Если нужен `apt dist-upgrade`, включи `system_upgrade_distribution: true`.

### Обновление конкретных пакетов

Вместо всех пакетов можно обновить только нужные:

```yaml
system_packages_to_update:
  - nginx
  - postgresql
  - python3
```

Если этот список не пустой, обновятся только эти пакеты, а не все.

### Закрепление пакетов

Если какие-то пакеты не должны обновляться, их можно закрепить:

```yaml
system_hold_packages:
  - kernel
  - kernel-headers
```

Эти пакеты будут закреплены через `dpkg --set-selections hold` и не будут обновляться.

### Очистка после обновления

```yaml
system_autoremove: true        # удалять неиспользуемые пакеты (по умолчанию true)
system_clean_apt_cache: true   # очищать кэш apt (по умолчанию true)
```

Обычно эти параметры лучше оставить включенными - они освобождают место на диске.

### Перезагрузка после обновления ядра

```yaml
system_reboot_after_kernel_update: false  # перезагружать если обновилось ядро (по умолчанию false)
system_reboot_delay: 300                  # задержка перед перезагрузкой в секундах (по умолчанию 300)
system_reboot_timeout: 600                # таймаут ожидания перезагрузки в секундах (по умолчанию 600)
```

Если обновилось ядро, система создаст файл `/var/run/reboot-required`. Роль может автоматически перезагрузить сервер после обновления, но по умолчанию это выключено для безопасности.

**Важно**: если включишь перезагрузку, убедись что у тебя есть доступ к серверу после перезагрузки (например, через IPMI или консоль провайдера), иначе можешь потерять доступ.

### Автоматические обновления (unattended-upgrades)

```yaml
system_unattended_upgrades: false  # настроить автоматические обновления (по умолчанию false)
```

По умолчанию выключено, потому что автоматические обновления могут сломать что-то важное. Включай только если понимаешь риски.

Если включишь, можно настроить детали:

```yaml
system_unattended_upgrades: true
system_unattended_upgrades_config:
  update_package_lists: "1"              # обновлять списки пакетов
  download_upgradeable_packages: "1"     # скачивать обновления
  autoremove_packages: "1"              # удалять неиспользуемые пакеты
  unattended_upgrade: "1"               # устанавливать обновления
  auto_fix_interrupted_dpkg: "1"        # исправлять прерванные установки
  minimal_steps: "1"                    # минимальные шаги
  mail_report: "on-change"              # отправлять email при изменениях
  mail_only_on_error: "false"           # отправлять email только при ошибках
  mail_to: "admin@example.com"          # email для уведомлений
  automatic_reboot: "false"              # автоматически перезагружать
  automatic_reboot_with_users: "false"  # перезагружать даже если есть пользователи
  automatic_reboot_time: "03:00"        # время для перезагрузки
```

Черный список пакетов для автоматических обновлений:

```yaml
system_unattended_upgrades_blacklist:
  - kernel
  - nginx
  - postgresql
```

Эти пакеты не будут обновляться автоматически.

### Проверки перед обновлением

```yaml
system_min_free_space_gb: 2              # минимум свободного места в GB (по умолчанию 2)
system_check_repositories: true          # проверять доступность репозиториев (по умолчанию true)
system_check_distribution_version: true  # проверять версию дистрибутива (по умолчанию true)
system_min_distribution_version: "11"    # минимальная версия (по умолчанию "11")
```

Роль проверяет что на диске достаточно места (минимум 2GB по умолчанию) и что репозитории доступны. Если что-то не так, обновление не начнётся.

### Логирование

```yaml
system_log_updates: true              # логировать результаты (по умолчанию true)
system_log_dir: "/var/log/ansible"   # директория для логов (по умолчанию /var/log/ansible)
system_show_upgrade_info: true       # показывать информацию об обновлениях (по умолчанию true)
```

Роль сохраняет лог обновлений в указанную директорию. В логе будет дата, хост, дистрибутив и список обновлённых пакетов.

### Проверка целостности

```yaml
system_check_package_integrity: true  # проверять целостность после обновления (по умолчанию true)
```

После обновления роль проверяет что все пакеты в порядке через `dpkg --audit` и `apt-get check`. Если что-то не так, выведет предупреждение.

### Автоматическое исправление ошибок

```yaml
system_auto_fix_on_error: true  # пытаться исправить ошибки автоматически (по умолчанию true)
```

Если при обновлении возникла ошибка, роль попытается исправить её через `dpkg --configure -a` и повторит обновление.

## Примеры использования

### Простое обновление всех пакетов

```yaml
- hosts: servers
  become: yes
  roles:
    - role: system-update
      vars:
        system_update_enabled: true
        system_upgrade_packages: true
```

### Обновление только security updates

```yaml
- hosts: servers
  become: yes
  roles:
    - role: system-update
      vars:
        system_security_only: true
```

### Обновление с перезагрузкой после обновления ядра

```yaml
- hosts: servers
  become: yes
  roles:
    - role: system-update
      vars:
        system_reboot_after_kernel_update: true
        system_reboot_delay: 600  # 10 минут задержка
```

### Обновление конкретных пакетов

```yaml
- hosts: servers
  become: yes
  roles:
    - role: system-update
      vars:
        system_packages_to_update:
          - nginx
          - postgresql-14
```

### Настройка автоматических обновлений

```yaml
- hosts: servers
  become: yes
  roles:
    - role: system-update
      vars:
        system_unattended_upgrades: true
        system_unattended_upgrades_config:
          mail_to: "admin@example.com"
          mail_report: "on-change"
          automatic_reboot: "false"
        system_unattended_upgrades_blacklist:
          - kernel
          - nginx
```

### Dist-upgrade (обновление дистрибутива)

```yaml
- hosts: servers
  become: yes
  roles:
    - role: system-update
      vars:
        system_upgrade_distribution: true
```

**Внимание**: dist-upgrade может обновить дистрибутив до новой версии (например, Debian 11 до 12), что может сломать что-то. Используй осторожно.

### Только проверка без обновления

```yaml
- hosts: servers
  become: yes
  roles:
    - role: system-update
      vars:
        system_update_enabled: false
```

Или используй теги для запуска только проверок:

```bash
ansible-playbook playbook.yml --tags check
```

## Теги

Роль использует теги для запуска отдельных частей:

- `check` - только проверки перед обновлением
- `update` - обновление пакетов
- `upgrade` - обновление пакетов
- `dist-upgrade` - обновление дистрибутива
- `security` - только security updates
- `autoremove` - удаление неиспользуемых пакетов
- `reboot` - перезагрузка
- `unattended` - настройка автоматических обновлений
- `log` - логирование
- `clean` - очистка кэша
- `integrity` - проверка целостности

Например, запустить только проверки:

```bash
ansible-playbook playbook.yml --tags check
```

Или обновить пакеты без перезагрузки:

```bash
ansible-playbook playbook.yml --tags update --skip-tags reboot
```

## Режим проверки (dry-run)

Можно запустить роль в режиме проверки, чтобы увидеть что бы изменилось без реального выполнения:

```bash
ansible-playbook playbook.yml --check --diff
```

Это покажет какие пакеты были бы обновлены, но ничего не изменит на сервере.

## Что происходит при ошибках

Если при обновлении возникла ошибка:

1. Роль выведет сообщение об ошибке
2. Если `system_auto_fix_on_error: true`, попытается исправить через `dpkg --configure -a`
3. После исправления попытается обновить снова
4. Если не помогло, завершится с ошибкой

В этом случае нужно зайти на сервер и исправить проблему вручную.

## Логи

Логи обновлений сохраняются в `/var/log/ansible/` (или в директории указанной в `system_log_dir`). Каждый файл содержит:

- Дата и время обновления
- Хост
- Дистрибутив
- Список обновлённых пакетов с версиями

Имя файла: `system-update-<timestamp>.log`

## Поддерживаемые дистрибутивы

- Debian 11 (Bullseye)
- Debian 12 (Bookworm)
- Debian 13 (Trixie)
- Ubuntu 20.04 (Focal)
- Ubuntu 22.04 (Jammy)
- Ubuntu 24.04 (Noble)

Роль проверяет версию дистрибутива перед обновлением и не будет работать на старых версиях.

## Важные замечания

1. **Перезагрузка**: если включишь `system_reboot_after_kernel_update: true`, сервер перезагрузится автоматически. Убедись что у тебя есть доступ к серверу после перезагрузки.

2. **Автоматические обновления**: `system_unattended_upgrades` по умолчанию выключено не просто так. Автоматические обновления могут сломать что-то важное. Включай только если понимаешь риски и протестировал на тестовом сервере.

3. **Dist-upgrade**: `system_upgrade_distribution: true` может обновить дистрибутив до новой версии. Это может сломать приложения. Используй осторожно.

4. **Резервные копии**: перед обновлением на продакшене сделай резервные копии важных данных.

5. **Тестирование**: сначала протестируй обновления на тестовом сервере, особенно если используешь dist-upgrade или автоматические обновления.

## Структура роли

Роль разделена на логические модули:

- `tasks/check.yml` - проверки перед обновлением
- `tasks/update.yml` - обновление пакетов
- `tasks/integrity.yml` - проверка целостности
- `tasks/logging.yml` - логирование
- `tasks/reboot.yml` - перезагрузка
- `tasks/unattended.yml` - настройка автоматических обновлений
- `handlers/main.yml` - обработчики событий
- `templates/50unattended-upgrades.j2` - шаблон конфигурации unattended-upgrades

Все параметры находятся в `defaults/main.yml`.

## Лицензия

ISC
