# Руководство по настройке dependency для Molecule

**Автор:** y.kossin  
**Дата:** январь 2026

---

## Обзор

Molecule поддерживает несколько способов указания источников ролей для dependency:

1. **Локальный каталог** - роли из файловой системы
2. **GitLab репозиторий** - роли из GitLab (публичные или приватные)
3. **Ansible Galaxy** - роли из официального репозитория
4. **Git репозиторий** - роли из любого Git репозитория

---

## Вариант 1: Локальный каталог

### Использование роли из локального каталога

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Локальная роль из файловой системы
- src: /home/kossin/OVR/jobs/kch/bindki-io/exchanger-iac/ansible/roles/firewall
  name: firewall
  scm: local
```

**Относительный путь:**

```yaml
---
# Относительный путь от текущей роли
- src: ../../firewall
  name: firewall
  scm: local
```

**Несколько локальных ролей:**

```yaml
---
- src: ../../firewall
  name: firewall
  scm: local

- src: ../../fail2ban
  name: fail2ban
  scm: local

- src: ../../libvirt-kvm
  name: libvirt-kvm
  scm: local
```

---

## Вариант 2: GitLab репозиторий (публичный)

### Использование роли из публичного GitLab репозитория

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Публичный GitLab репозиторий
- src: git+https://gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: main  # или master, или тег v1.0.0
```

**С указанием ветки:**

```yaml
---
- src: git+https://gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: develop  # ветка develop
```

**С указанием тега:**

```yaml
---
- src: git+https://gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: v1.2.3  # конкретный тег
```

---

## Вариант 3: GitLab репозиторий (приватный)

### Использование роли из приватного GitLab репозитория

**С токеном доступа:**

```yaml
---
# Приватный GitLab репозиторий с токеном
- src: git+https://oauth2:YOUR_TOKEN@gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: main
```

**С пользователем и токеном:**

```yaml
---
# С указанием пользователя и токена
- src: git+https://username:TOKEN@gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: main
```

**С переменной окружения:**

```yaml
---
# Токен из переменной окружения
- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: main
```

**Использование:**

```bash
export GITLAB_TOKEN=your_token_here
molecule test
```

---

## Вариант 4: GitLab репозиторий (SSH)

### Использование SSH для доступа к GitLab

```yaml
---
# Через SSH (требует настройки SSH ключей)
- src: git@gitlab.com:your-group/ansible-role-firewall.git
  name: firewall
  version: main
  scm: git
```

---

## Вариант 5: Ansible Galaxy

### Использование роли из Ansible Galaxy

```yaml
---
# Роль из Ansible Galaxy
- name: geerlingguy.docker
  version: 7.0.0

- name: geerlingguy.security
  version: 2.0.0
```

**Без указания версии (последняя):**

```yaml
---
- name: geerlingguy.docker
```

---

## Вариант 6: Комбинация источников

### Использование нескольких источников одновременно

```yaml
---
# Локальные роли
- src: ../../firewall
  name: firewall
  scm: local

- src: ../../fail2ban
  name: fail2ban
  scm: local

# Роль из GitLab
- src: git+https://gitlab.com/your-group/ansible-role-custom.git
  name: custom-role
  version: main

# Роль из Galaxy
- name: geerlingguy.docker
  version: 7.0.0
```

---

## Конфигурация Molecule

**Файл:** `molecule/default/molecule.yml`

```yaml
dependency:
  name: galaxy
  enabled: true
  options:
    role-file: requirements.yml
    requirements-file: requirements.yml
```

---

## Примеры для проекта bindki-io

### Использование всех ролей из локального каталога

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Все роли из локального каталога exchanger-iac/ansible/roles
- src: ../../firewall
  name: firewall
  scm: local

- src: ../../fail2ban
  name: fail2ban
  scm: local

- src: ../../libvirt-kvm
  name: libvirt-kvm
  scm: local

- src: ../../host-optimization
  name: host-optimization
  scm: local
```

### Использование ролей из GitLab

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Роли из GitLab репозитория
- src: git+https://gitlab.com/bindki-io/ansible-roles/firewall.git
  name: firewall
  version: main

- src: git+https://gitlab.com/bindki-io/ansible-roles/fail2ban.git
  name: fail2ban
  version: main
```

---

## Установка ролей вручную

Если нужно установить роли вручную перед запуском Molecule:

```bash
cd /home/kossin/OVR/jobs/kch/bindki-io/exchanger-iac/ansible/roles/system-update

# Установить роли из requirements.yml
ansible-galaxy install -r molecule/default/requirements.yml -p molecule/default/roles

# Или в стандартное место
ansible-galaxy install -r molecule/default/requirements.yml
```

---

## Переменные окружения для токенов

Для приватных репозиториев можно использовать переменные окружения:

```bash
# Установить токен
export GITLAB_TOKEN=your_token_here

# Запустить тесты
molecule test
```

Или создать файл `.env`:

```bash
# .env
GITLAB_TOKEN=your_token_here
```

---

## Проверка установки ролей

```bash
# Проверить установленные роли
ansible-galaxy list

# Проверить роли в molecule
ls -la molecule/default/roles/
```

---

## Устранение проблем

### Ошибка: "Role not found"

**Проблема:** Роль не найдена по указанному пути.

**Решение:** Проверить путь к роли:
```bash
ls -la /path/to/role
```

### Ошибка: "Authentication failed" (GitLab)

**Проблема:** Не удается получить доступ к приватному репозиторию.

**Решение:** 
1. Проверить токен доступа
2. Убедиться, что токен имеет права на чтение репозитория
3. Использовать правильный формат URL с токеном

### Ошибка: "Version not found" (GitLab)

**Проблема:** Указанная ветка или тег не существует.

**Решение:** Проверить доступные ветки и теги в репозитории.

---

## Рекомендации

1. **Для локальной разработки:** Используйте локальные роли (`scm: local`)
2. **Для CI/CD:** Используйте GitLab репозитории с токенами
3. **Для публичных ролей:** Используйте Ansible Galaxy
4. **Для приватных ролей:** Используйте GitLab с токенами доступа

---

**Последнее обновление:** январь 2026
