
---

# infisical-secrets

Ansible-роль для получения секретов из [Infisical](https://infisical.com/) с поддержкой **нескольких проектов** и **нескольких окружений** (prod / staging / dev / …).

## Простой пример

Указываем проект — получаем пароли в виде словаря:

```yaml
# group_vars/all.yml
infisical_api_url: "https://infisical.example.com/"
infisical_token_env_var: "INFISICAL_TOKEN"

infisical_projects:
  - id: "63b94a55-3875-4ad1-bab7-72b0de685b76"
    name: "backend"
    environments: [prod]
    secret_specs:
      - path: "/"
        keys: []   # пустой список = получить ВСЕ ключи
```

```yaml
# playbook.yml
- hosts: all
  roles:
    - role: infisical-secrets

  tasks:
    # Обратиться к конкретному ключу
    - debug:
        msg: "Пароль БД: {{ infisical_all_secrets.backend.prod.DB_PASSWORD }}"

    # Получить все ключи в виде массива (списка имён)
    - debug:
        msg: "Все ключи: {{ infisical_all_secrets.backend.prod.keys() | list }}"

    # Получить все пары ключ=значение в виде словаря
    - debug:
        var: infisical_all_secrets.backend.prod
```

Запуск:
```bash
INFISICAL_TOKEN=st.xxxx ansible-playbook playbook.yml
```

Результат (`infisical_all_secrets.backend.prod`) будет выглядеть так:
```json
{
  "DB_PASSWORD": "s3cr3t",
  "API_KEY": "abc123",
  "REDIS_URL": "redis://localhost:6379"
}
```

---

## Результирующие факты

| Факт | Описание |
|------|----------|
| `infisical_all_secrets[project][env][KEY]` | Вложенный dict: проект → окружение → ключ |

## Пример с несколькими проектами

```yaml
infisical_projects:
  - id: "63b94a55-3875-4ad1-bab7-72b0de685b76"
    name: "backend"
    environments: [prod, staging]
    secret_specs:
      - path: "/"
        keys: []

  - id: "be5f257d-1b4d-431b-a1b8-c968500c7b47"
    name: "frontend"
    environments: [prod]
    secret_specs:
      - path: "/"
        keys: []
      - path: "/api"
        keys:
          - API_KEY
          - API_SECRET
```

## Использование секретов

```yaml
# Весь prod из проекта backend
- debug:
    var: infisical_all_secrets.backend.prod

# Отдельный ключ
- debug:
    msg: "{{ infisical_all_secrets.backend.prod.DATABASE_URL }}"
```

## Переменные

| Переменная | По умолчанию                     | Описание |
|------------|----------------------------------|----------|
| `infisical_api_url` | `https://infisical.example.com/` | URL Infisical API |
| `infisical_token_env_var` | `INFISICAL_TOKEN`                | ENV переменная с токеном |
| `infisical_projects` | `[]`                             | Список проектов (основной параметр) |
| `infisical_validate_certs` | `true`                           | Проверять TLS |
| `infisical_fail_on_missing` | `true`                           | Фейлить при отсутствии явно запрошенных ключей |
| `infisical_distribute_facts` | `true`                           | Распределять факты на все хосты |
| `infisical_no_log` | `true`                           | Скрывать значения в выводе |
| `infisical_debug` | `false`                          | Подробный debug вывод |
| `infisical_show_summary` | `true`                           | Показывать сводку ключей |

## Обратная совместимость (legacy)

Если `infisical_projects` не задан, но заданы старые переменные — они автоматически конвертируются:

```yaml
infisical_project_id: "abc123"
infisical_environment: "prod"
infisical_secret_specs:
  - path: "/"
    keys: []
```
