# Быстрая настройка dependency для Molecule

**Автор:** y.kossin

---

## ✅ Да, можно использовать:

1. **Локальный каталог** - роли из файловой системы
2. **GitLab репозиторий** - публичные или приватные
3. **Ansible Galaxy** - официальный репозиторий

---

## Быстрая настройка

### Вариант 1: Локальные роли

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Локальные роли из каталога
- src: ../../firewall
  name: firewall
  scm: local

- src: ../../fail2ban
  name: fail2ban
  scm: local
```

**Или скопировать готовый пример:**
```bash
cp molecule/default/requirements-local.yml.example molecule/default/requirements.yml
# Отредактировать и раскомментировать нужные роли
```

---

### Вариант 2: GitLab репозиторий

**Файл:** `molecule/default/requirements.yml`

```yaml
---
# Публичный репозиторий
- src: git+https://gitlab.com/your-group/ansible-role-firewall.git
  name: firewall
  version: main

# Приватный репозиторий (с токеном)
- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/your-group/ansible-role-fail2ban.git
  name: fail2ban
  version: main
```

**Использование:**
```bash
export GITLAB_TOKEN=your_token_here
molecule test
```

**Или скопировать готовый пример:**
```bash
cp molecule/default/requirements-gitlab.yml.example molecule/default/requirements.yml
# Отредактировать URL репозиториев
```

---

### Вариант 3: Только текущая роль (по умолчанию)

**Файл:** `molecule/default/requirements.yml` - оставить пустым или не создавать

**Или отключить dependency:**
```yaml
# molecule/default/molecule.yml
dependency:
  name: galaxy
  enabled: false
```

---

## Примеры для проекта bindki-io

### Все локальные роли

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

- src: ../../host-optimization
  name: host-optimization
  scm: local
```

### Роли из GitLab

```yaml
---
- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/bindki-io/ansible-roles/firewall.git
  name: firewall
  version: main

- src: git+https://oauth2:{{ lookup('env', 'GITLAB_TOKEN') }}@gitlab.com/bindki-io/ansible-roles/fail2ban.git
  name: fail2ban
  version: main
```

---

## Документация

- [DEPENDENCY_GUIDE.md](DEPENDENCY_GUIDE.md) - полное руководство
- [DEPENDENCY_EXAMPLES.md](DEPENDENCY_EXAMPLES.md) - детальные примеры
- [requirements-local.yml.example](default/requirements-local.yml.example) - пример для локальных ролей
- [requirements-gitlab.yml.example](default/requirements-gitlab.yml.example) - пример для GitLab

---

**Последнее обновление:** январь 2026
