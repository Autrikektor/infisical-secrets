# infisical-secrets

Ansible role for fetching secrets from [Infisical](https://infisical.com/) with support for **multiple projects** and **multiple environments** (prod / staging / dev / …).

## Quick example

Specify a project ID and get all its secrets as a dictionary:

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
        keys: []   # empty = fetch ALL keys
```

```yaml
# playbook.yml
- hosts: all
  roles:
    - role: infisical-secrets

  tasks:
    # Access individual key
    - debug:
        msg: "DB password: {{ infisical_all_secrets.backend.prod.DB_PASSWORD }}"

    # Get all key names as a list (array)
    - debug:
        msg: "All secret keys: {{ infisical_all_secrets.backend.prod.keys() | list }}"

    # Get all key=value pairs as a dict
    - debug:
        var: infisical_all_secrets.backend.prod
```

Run:
```bash
INFISICAL_TOKEN=st.xxxx ansible-playbook playbook.yml
```

Result (`infisical_all_secrets.backend.prod`) will look like:
```json
{
  "DB_PASSWORD": "s3cr3t",
  "API_KEY": "abc123",
  "REDIS_URL": "redis://localhost:6379"
}
```

---

## Result facts

| Fact | Description |
|------|-------------|
| `infisical_all_secrets[project][env][KEY]` | Nested dict: project → environment → key |

## Multi-project example

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

## Using secrets

```yaml
# All prod secrets from project "backend"
- debug:
    var: infisical_all_secrets.backend.prod

# A single key
- debug:
    msg: "{{ infisical_all_secrets.backend.prod.DATABASE_URL }}"
```

## Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `infisical_api_url` | `https://infisical.example.com/` | Infisical API URL |
| `infisical_token_env_var` | `INFISICAL_TOKEN` | ENV variable holding the token |
| `infisical_projects` | `[]` | List of projects (main parameter) |
| `infisical_validate_certs` | `true` | Validate TLS certificates |
| `infisical_fail_on_missing` | `true` | Fail if explicitly requested keys are missing |
| `infisical_distribute_facts` | `true` | Distribute facts to all play hosts |
| `infisical_no_log` | `true` | Hide secret values in output |
| `infisical_debug` | `false` | Verbose debug output |
| `infisical_show_summary` | `true` | Show key summary after fetch |

## Legacy (single-project) backwards compatibility

If `infisical_projects` is not set but old variables are defined, they are auto-converted:

```yaml
infisical_project_id: "abc123"
infisical_environment: "prod"
infisical_secret_specs:
  - path: "/"
    keys: []
```
