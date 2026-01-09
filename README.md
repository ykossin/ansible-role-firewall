# Ansible Role: Firewall

Роль для установки и настройки firewall (ufw) на Debian/Ubuntu системах.

## Описание

Эта роль устанавливает и настраивает ufw (Uncomplicated Firewall) для защиты сервера:
- Установка ufw
- Настройка политик по умолчанию
- Настройка правил для портов и IP адресов
- Настройка логирования
- Управление IPv6

## Требования

- Debian 11+ или Ubuntu 20.04+
- Доступ root/sudo
- Ansible >= 2.9

## Переменные роли

См. `defaults/main.yml` для всех доступных переменных.

### Основные переменные

```yaml
# Включить firewall
firewall_enabled: true

# Состояние firewall
firewall_state: started
firewall_enabled_on_boot: true

# Политики по умолчанию
firewall_default_incoming: "deny"
firewall_default_outgoing: "allow"

# Разрешенные порты
firewall_allowed_ports:
  - { port: "22/tcp", comment: "SSH" }
  - { port: "80/tcp", comment: "HTTP" }
  - { port: "443/tcp", comment: "HTTPS" }

# Настройки логирования
firewall_logging: "low"  # off, low, medium, full
```

## Использование

### Базовая настройка

```yaml
- hosts: all
  become: true
  roles:
    - firewall
```

### С настройкой портов

```yaml
- hosts: all
  become: true
  vars:
    firewall_allowed_ports:
      - { port: "22/tcp", comment: "SSH" }
      - { port: "80/tcp", comment: "HTTP" }
      - { port: "443/tcp", comment: "HTTPS" }
      - { port: "16509/tcp", comment: "libvirt" }
  roles:
    - firewall
```

### С разрешением IP адресов

```yaml
- hosts: all
  become: true
  vars:
    firewall_allowed_ports:
      - { port: "22/tcp", comment: "SSH" }
    firewall_allowed_ips:
      - { ip: "10.0.0.0/8", comment: "Internal network" }
      - { ip: "192.168.1.0/24", comment: "LAN" }
  roles:
    - firewall
```

### С блокировкой IP

```yaml
- hosts: all
  become: true
  vars:
    firewall_blocked_ips:
      - { ip: "1.2.3.4", comment: "Blocked IP" }
  roles:
    - firewall
```

## Теги

- `firewall` - все задачи firewall
- `ufw` - задачи ufw
- `packages` - установка пакетов
- `policy` - настройка политик
- `rules` - настройка правил
- `logging` - настройка логирования
- `service` - управление сервисом
- `check` - проверка статуса
- `reset` - сброс правил

## Примеры использования тегов

```bash
# Только проверка статуса
ansible-playbook playbook.yml --tags check

# Настройка правил без включения
ansible-playbook playbook.yml --tags rules --skip-tags service

# Сброс и перенастройка
ansible-playbook playbook.yml --tags reset,policy,rules
```

## Безопасность

### ⚠️ ВАЖНО:

1. **SSH доступ:** Убедитесь, что SSH порт (22) разрешен ДО применения firewall, иначе можно потерять доступ к серверу!

2. **Тестирование:** Протестируйте правила на тестовом сервере перед применением на production

3. **Резервное подключение:** Имейте альтернативный способ доступа к серверу (консоль, IPMI, KVM)

4. **Постепенное применение:** Применяйте правила постепенно, проверяя доступность сервисов

## Troubleshooting

### Проблема: Потерян доступ по SSH

Если заблокировали SSH случайно:
```bash
# Через консоль или IPMI:
ufw allow 22/tcp
ufw reload
```

### Проверка правил

```bash
# Статус firewall
ufw status verbose

# Логи firewall
tail -f /var/log/ufw.log

# Проверка портов
ss -tlnp | grep LISTEN
```

## Лицензия

ISC License

---

**Последнее обновление:** январь 2026
