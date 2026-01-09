# Ansible Role: Firewall

[![Ansible Galaxy](https://img.shields.io/badge/galaxy-ykossin.firewall-blue.svg)](https://galaxy.ansible.com/ykossin/firewall)
[![License](https://img.shields.io/badge/license-ISC-green.svg)](LICENSE)

Роль для установки и настройки UFW (Uncomplicated Firewall) на Debian/Ubuntu системах.

## Описание

Эта роль автоматизирует установку и настройку UFW firewall для защиты сервера:

- ✅ Установка и настройка UFW
- ✅ Настройка политик по умолчанию (incoming, outgoing, routed)
- ✅ Управление правилами для портов и IP адресов
- ✅ Настройка логирования
- ✅ Управление IPv6
- ✅ Проверка SSH доступа перед включением firewall
- ✅ Поддержка идемпотентности (правила не дублируются)
- ✅ Поддержка check mode (dry-run)
- ✅ Molecule тесты
- ✅ Подробная обработка ошибок

## Требования

- **ОС:** Debian 11+ (Bullseye, Bookworm, Trixie) или Ubuntu 20.04+ (Focal, Jammy, Noble)
- **Ansible:** >= 2.9
- **Права:** root или sudo доступ
- **Python:** Python 3 (обычно предустановлен)

## Установка

### Из Ansible Galaxy

```bash
ansible-galaxy role install ykossin.firewall
```

### Из GitHub

```bash
ansible-galaxy role install git+https://github.com/ykossin/ansible-role-firewall.git
```

### Из локального репозитория

```bash
ansible-galaxy role install /path/to/ansible-role-firewall
```

## Переменные роли

### Основные переменные

| Переменная | По умолчанию | Описание |
|------------|--------------|----------|
| `firewall_enabled` | `true` | Включить установку и настройку firewall |
| `firewall_type` | `"ufw"` | Тип firewall (поддерживается только ufw) |
| `firewall_state` | `"enabled"` | Состояние firewall: `enabled`, `disabled`, `reloaded`, `reset` |
| `firewall_enabled_on_boot` | `true` | Включить firewall при загрузке системы |

### Политики по умолчанию

| Переменная | По умолчанию | Описание |
|------------|--------------|----------|
| `firewall_default_incoming` | `"deny"` | Политика для входящих соединений: `allow`, `deny`, `reject` |
| `firewall_default_outgoing` | `"allow"` | Политика для исходящих соединений: `allow`, `deny`, `reject` |
| `firewall_default_forward` | `"deny"` | Политика для форвардинга: `allow`, `deny`, `reject` |
| `firewall_default_routed` | `"deny"` | Политика для маршрутизации: `allow`, `deny`, `reject` |

### Правила для портов

```yaml
firewall_allowed_ports:
  - {port: "22/tcp", comment: "SSH"}
  - {port: "80/tcp", comment: "HTTP"}
  - {port: "443/tcp", comment: "HTTPS"}
  - {port: "3306/tcp", comment: "MySQL"}
```

**Формат:**
- `port`: номер порта и протокол (например, `"22/tcp"`, `"53/udp"`)
- `comment`: комментарий к правилу (опционально)

### Правила для IP адресов

```yaml
# Разрешить доступ с определенных IP/сетей
firewall_allowed_ips:
  - {ip: "10.0.0.0/8", comment: "Internal network"}
  - {ip: "192.168.1.0/24", comment: "LAN"}
  - {ip: "203.0.113.0/24", comment: "Office network"}

# Заблокировать определенные IP
firewall_blocked_ips:
  - {ip: "1.2.3.4", comment: "Blocked IP"}
  - {ip: "5.6.7.8", comment: "Suspicious IP"}
```

### Настройки логирования

| Переменная | По умолчанию | Описание |
|------------|--------------|----------|
| `firewall_logging` | `"low"` | Уровень логирования: `off`, `low`, `medium`, `full`, `on` |
| `firewall_log_ufw` | `false` | Логирование удаленных соединений |
| `firewall_ipv6` | `false` | Включить поддержку IPv6 |

### Дополнительные настройки

| Переменная | По умолчанию | Описание |
|------------|--------------|----------|
| `firewall_reset` | `false` | Сбросить все правила перед настройкой (⚠️ осторожно!) |
| `firewall_skip_ssh_check` | `false` | Пропустить проверку SSH порта (⚠️ не рекомендуется!) |
| `firewall_raw_rules` | `[]` | Raw правила ufw (продвинутые пользователи) |

### Примеры raw правил

```yaml
firewall_raw_rules:
  - {rule: "-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT", comment: "Allow ping"}
```

## Примеры использования

### Базовый пример

```yaml
- hosts: servers
  become: true
  roles:
    - role: ykossin.firewall
      vars:
        firewall_allowed_ports:
          - {port: "22/tcp", comment: "SSH"}
          - {port: "80/tcp", comment: "HTTP"}
          - {port: "443/tcp", comment: "HTTPS"}
```

### Пример с кастомными настройками

```yaml
- hosts: web_servers
  become: true
  roles:
    - role: ykossin.firewall
      vars:
        firewall_default_incoming: "deny"
        firewall_default_outgoing: "allow"
        firewall_logging: "medium"
        firewall_ipv6: true
        firewall_allowed_ports:
          - {port: "22/tcp", comment: "SSH"}
          - {port: "80/tcp", comment: "HTTP"}
          - {port: "443/tcp", comment: "HTTPS"}
          - {port: "3306/tcp", comment: "MySQL"}
        firewall_allowed_ips:
          - {ip: "10.0.0.0/8", comment: "Internal network"}
        firewall_blocked_ips:
          - {ip: "192.0.2.100", comment: "Blocked IP"}
```

### Пример с проверкой (dry-run)

```yaml
- hosts: servers
  become: true
  roles:
    - role: ykossin.firewall
```

Запуск в режиме проверки:
```bash
ansible-playbook playbook.yml --check
```

### Отключение firewall

```yaml
- hosts: servers
  become: true
  roles:
    - role: ykossin.firewall
      vars:
        firewall_state: "disabled"
```

## Безопасность

### ⚠️ Важные предупреждения

1. **SSH доступ:** Роль автоматически проверяет наличие порта 22 в разрешенных портах перед включением firewall. Если SSH порт не найден, роль остановит выполнение с критическим предупреждением.

2. **Сброс правил:** Использование `firewall_reset: true` удалит все существующие правила. Используйте с осторожностью!

3. **Пропуск проверки SSH:** Использование `firewall_skip_ssh_check: true` может привести к потере доступа к серверу. Используйте только если уверены в альтернативном способе доступа (консоль, IPMI, KVM).

### Рекомендации

- Всегда проверяйте конфигурацию в режиме `--check` перед применением
- Убедитесь, что SSH порт (22) включен в `firewall_allowed_ports`
- Используйте минимально необходимые правила
- Регулярно проверяйте логи: `journalctl -u ufw` или `/var/log/ufw.log`

## Тестирование

Роль включает Molecule тесты для проверки функциональности:

```bash
cd roles/firewall
molecule test
```

Тесты проверяют:
- Установку ufw
- Настройку политик
- Добавление правил для портов
- Идемпотентность (правила не дублируются)
- Включение firewall

## Структура роли

```
firewall/
├── defaults/
│   └── main.yml          # Переменные по умолчанию
├── handlers/
│   └── main.yml          # Handlers для перезагрузки ufw
├── meta/
│   └── main.yml          # Метаданные для Galaxy
├── tasks/
│   ├── main.yml          # Главный файл задач
│   ├── validation.yml    # Валидация переменных
│   ├── installation.yml  # Установка ufw
│   ├── policies.yml      # Настройка политик
│   ├── logging.yml       # Настройка логирования
│   ├── rules-ports.yml   # Правила для портов
│   ├── rules-ips.yml     # Правила для IP адресов
│   ├── rules-raw.yml     # Raw правила
│   ├── ssh-check.yml     # Проверка SSH доступа
│   ├── service.yml       # Управление состоянием
│   └── status.yml        # Проверка статуса
├── tests/
│   ├── inventory         # Тестовый inventory
│   └── test.yml          # Тестовый playbook
└── README.md             # Документация
```

## Идемпотентность

Роль полностью идемпотентна:

- ✅ Правила для портов не дублируются
- ✅ Правила для IP адресов не дублируются
- ✅ Raw правила проверяются перед добавлением
- ✅ Политики применяются только при необходимости

## Check Mode

Роль поддерживает режим проверки (`--check`):

- Все задачи выполняются в режиме dry-run
- Проверяется наличие ufw перед выполнением задач
- Выводится информация о планируемых изменениях

## Обработка ошибок

Роль включает подробную обработку ошибок:

- ✅ Информативные сообщения об ошибках
- ✅ Проверка прав доступа
- ✅ Проверка доступности репозиториев
- ✅ Проверка статуса ufw
- ✅ Рекомендации по устранению проблем

## Зависимости

Роль не имеет зависимостей от других ролей.

## Поддерживаемые платформы

- Debian 11 (Bullseye)
- Debian 12 (Bookworm)
- Debian 13 (Trixie)
- Ubuntu 20.04 (Focal)
- Ubuntu 22.04 (Jammy)
- Ubuntu 24.04 (Noble)

## Лицензия

ISC

## Автор

**y.kossin**

- GitHub: [@ykossin](https://github.com/ykossin)
- Ansible Galaxy: [ykossin.firewall](https://galaxy.ansible.com/ykossin/firewall)

## История версий

### v1.1.0 (2026-01-09)

- ✅ Исправлен `firewall_state` с `started` на `enabled` (правильное значение для модуля ufw)
- ✅ Добавлены Molecule тесты
- ✅ Улучшена структура задач (разделение на логические файлы)
- ✅ Добавлена поддержка check mode
- ✅ Улучшена обработка ошибок
- ✅ Добавлена проверка идемпотентности для всех типов правил
- ✅ Улучшена проверка SSH доступа
- ✅ Исправлены ошибки линтера

### v1.0.0

- Первый релиз

## Поддержка

Если вы нашли ошибку или у вас есть предложения по улучшению, пожалуйста, создайте issue на GitHub:

https://github.com/ykossin/ansible-role-firewall/issues

## Благодарности

- Команде разработчиков UFW
- Сообществу Ansible
- Всем контрибьюторам
