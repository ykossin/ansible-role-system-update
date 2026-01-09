# Примеры использования dependency в Molecule

**Автор:** y.kossin  
**Дата:** январь 2026

---

## Быстрый старт

### Вариант 1: Только локальная роль (по умолчанию)

**Файл:** `molecule/default/requirements.yml` - оставить пустым или не создавать

Текущая роль `system-update` используется локально, dependency можно отключить:

```yaml
# molecule/default/molecule.yml
dependency:
  name: galaxy
  enabled: false  # Текущая роль используется локально
```

---

### Вариант 2: Локальные роли из каталога

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Локальные роли из каталога exchanger-iac/ansible/roles
- src: ../../firewall
  name: firewall
  scm: local

- src: ../../fail2ban
  name: fail2ban
  scm: local
```

**Конфигурация:** `molecule/default/molecule.yml`

```yaml
dependency:
  name: galaxy
  enabled: true
  options:
    role-file: requirements.yml
    requirements-file: requirements.yml
```

---

### Вариант 3: Роли из GitLab репозитория

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Публичный репозиторий
- src: git+https://gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: main

# Приватный репозиторий с токеном
- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/your-group/ansible-role-fail2ban.git
  name: fail2ban
  version: main
```

**Использование:**

```bash
export GITLAB_TOKEN=your_token_here
molecule test
```

---

## Детальные примеры

### Пример 1: Все локальные роли проекта

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Все роли из локального каталога
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

- src: ../../proxmox-install
  name: proxmox-install
  scm: local
```

---

### Пример 2: GitLab репозитории (публичные)

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Роли из публичных GitLab репозиториев
- src: git+https://gitlab.com/bindki-io/ansible-roles/firewall.git
  name: firewall
  version: main

- src: git+https://gitlab.com/bindki-io/ansible-roles/fail2ban.git
  name: fail2ban
  version: main

- src: git+https://gitlab.com/bindki-io/ansible-roles/libvirt-kvm.git
  name: libvirt-kvm
  version: v1.0.0  # конкретный тег
```

---

### Пример 3: GitLab репозитории (приватные)

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Роли из приватных GitLab репозиториев
# Токен из переменной окружения
- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/bindki-io/ansible-roles/firewall.git
  name: firewall
  version: main

- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/bindki-io/ansible-roles/fail2ban.git
  name: fail2ban
  version: main
```

**Создать токен в GitLab:**

1. Перейти в GitLab → Settings → Access Tokens
2. Создать токен с правами `read_repository`
3. Использовать токен:

```bash
export GITLAB_TOKEN=glpat-xxxxxxxxxxxxx
molecule test
```

---

### Пример 4: Комбинация источников

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Локальные роли
- src: ../../firewall
  name: firewall
  scm: local

- src: ../../fail2ban
  name: fail2ban
  scm: local

# Роли из GitLab
- src: git+https://gitlab.com/bindki-io/ansible-roles/libvirt-kvm.git
  name: libvirt-kvm
  version: main

# Роли из Galaxy
- name: geerlingguy.docker
  version: 7.0.0
```

---

## Настройка для проекта bindki-io

### Использование всех локальных ролей

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

- src: ../../proxmox-install
  name: proxmox-install
  scm: local
```

### Использование ролей из GitLab

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Роли из GitLab репозиториев проекта bindki-io
- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/bindki-io/ansible-roles/firewall.git
  name: firewall
  version: main

- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/bindki-io/ansible-roles/fail2ban.git
  name: fail2ban
  version: main

- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/bindki-io/ansible-roles/libvirt-kvm.git
  name: libvirt-kvm
  version: main
```

---

## Проверка установки ролей

```bash
# Проверить установленные роли
ansible-galaxy list

# Проверить роли в molecule
ls -la molecule/default/roles/

# Установить роли вручную
ansible-galaxy install -r molecule/default/requirements.yml
```

---

## Устранение проблем

### Ошибка: "Role not found" (локальная роль)

**Проблема:** Путь к локальной роли неверный.

**Решение:** Проверить путь:
```bash
ls -la ../../firewall
```

### Ошибка: "Authentication failed" (GitLab)

**Проблема:** Неверный токен или нет доступа.

**Решение:**
1. Проверить токен: `echo $GITLAB_TOKEN`
2. Убедиться, что токен имеет права `read_repository`
3. Проверить формат URL

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
