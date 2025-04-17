# Configuration Management

All configuration settings should be stored in [environment variables](https://12factor.net/config).

## Environment Variables

Since we strictly use environment variables for configuration,
each project must include a `.env.example` file in the root directory.
This file serves as a reference for required configuration variables.
The actual `.env` file, which contains sensitive information,
must be added to `.gitignore` to prevent accidental commits to version control.

## `.env` File Format

Each line should follow the **KEY=value** format:
Keys must be **UPPERCASE** and should start with the **PROJECT NAME** to avoid conflicts.

### Comments in `.env` Files

Comments should be prefixed with `#`.
Everything after `#` on a line is ignored by the parser.
Use comments to explain configuration variables where necessary.

### Secrets

Secrets and sensitive data will start with `SECRET_`.
This way, we can obtain secrets from secure secret management tools like
[HashiCorp Vault](https://developer.hashicorp.com/vault).

### Example:

```env
EZEX_GATEWAY_REDIS_HOST="localhost"
EZEX_GATEWAY_REDIS_PORT=6379

EZEX_GATEWAY_REDIS_PASSWORD="${SECRET_EZEX_GATEWAY_REDIS_PASSWORD}"
```

## Limitations

Environment variables are simple key-value pairs and are not suitable for storing complex data types like arrays or objects.
To work around this limitation, you can store data as a JSON string and parse it in your application.

```env
PROXIER_PROXY='[ \
  {"endpoint": "foo", "destination_url": "https://api.example.com"} \
]'
```
