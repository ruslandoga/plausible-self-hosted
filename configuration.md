Plausible is configured with environment variables, by default supplied via [<kbd>plausible-conf.env</kbd>](https://github.com/plausible/hosting/blob/master/plausible-conf.env) [env_file.](https://github.com/plausible/hosting/blob/bb6decee4d33ccf84eb235b6053443a01498db53/docker-compose.yml#L38-L39) They are read at startup in [`config/runtime.exs`](https://github.com/plausible/analytics/blob/master/config/runtime.exs)

In the [container image](https://hub.docker.com/r/plausible/analytics) this script is located at `/app/releases/0.0.1/runtime.exs` and you can mount a custom one if, for example, you want to configure some option which doesn't have an environment varible in the default script.

Note that if you start a container with one set of ENV vars and then update the ENV vars and restart the container, they won't take effect due to the immutable nature of the containers. The container needs to be recreated.

Here's the minimal <kbd>plausible-conf.env</kbd>

```env
BASE_URL=https://who.copycat.fun
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

And here's <kbd>plausible-conf.env</kbd> with extra configuration

```env
BASE_URL=https://who.copycat.fun
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
PORT=8000
MAXMIND_LICENSE_KEY=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
MAXMIND_EDITION=GeoLite2-Country
GOOGLE_CLIENT_ID=140927866833-002gqg48rl4iku76lbkk0qhu0i0m7bia.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=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
MAILER_NAME=Plausible Who
MAILER_EMAIL=plausible@who.copycat.fun
SENTRY_DSN=https://7f16d5d6ee70465789e082bd09481556@o1012425.ingest.sentry.io/6643873
DISABLE_REGISTRATION=invite_only
```

Here're the currently supported ENV vars:

### Required

#### `BASE_URL`

Configures the URL to use in link generation, doesn't have any defaults and needs to be provided in the ENV vars

<sub><kbd>plausible-conf.env</kbd></sub>
```env
BASE_URL=https://who.copycat.fun
```

---

#### `SECRET_KEY_BASE`

Configures the secret used for sessions in the dashboard, doesn't have any defaults and needs to be provided in the ENV vars, can be generated with `openssl rand -base64 48`

<sub><kbd>console</kbd></sub>
```console
$ openssl rand -base64 48
ouVdiHkQ11wdrafe4GzIuAPjcdWjFduIse6rBd3HtF99d34b6ivlPaS2Yh4I7ImE
```

<sub><kbd>plausible-conf.env</kbd></sub>
```env
SECRET_KEY_BASE=ouVdiHkQ11wdrafe4GzIuAPjcdWjFduIse6rBd3HtF99d34b6ivlPaS2Yh4I7ImE
``````

> ⚠️ Don't use this exact value or someone would be able to sign a cookie with `user_id=1` and log in as the admin.

### Registration

#### `DISABLE_REGISTRATION`

Default: `true`

---

#### `ENABLE_EMAIL_VERIFICATION`

Default: `false`

### Web

#### `LISTEN_IP`

Default: `0.0.0.0`

Configures the IP address to bind the listen socket for the web server. Note that setting it to `127.0.0.1` in a container would make the web server unavailable from outside the container.

---

#### `PORT`

Default: `8000`

Configures the port to bind the listen socket for the web server.

### Database

#### `DATABASE_URL`

Default: `postgres://postgres:postgres@plausible_db:5432/plausible_db`

Configures the URL for PostgreSQL database.

---

#### `CLICKHOUSE_DATABASE_URL`

Default: `http://plausible_events_db:8123/plausible_events_db`

Configures the URL for ClickHouse database.

---

#### `ECTO_IPV6`

---

#### `DATABASE_CACERTFILE`

---

#### `ECTO_CH_IPV6`

---

#### `CLICKHOUSE_CACERTFILE`

### Google

#### `GOOGLE_CLIENT_ID`

---

#### `GOOGLE_CLIENT_SECRET`

### Locations

#### `IP_GEOLOCATION_DB`

Default: `/app/lib/plausible-0.0.1/priv/geodb/dbip-country.mmdb.gz`

Defaults to the one shipped within the container image.

This database is used to lookup GeoName IDs for IP addresses.

---

#### `GEONAMES_SOURCE_FILE`

Default: `/app/lib/location-0.1.0/priv/geonames.lite.csv`

Defaults to the one shipped within the container image.

This file is used to turn GeoName IDs into human readable string for display on the dashboard.

---

#### `MAXMIND_LICENSE_KEY`

If set, this ENV variable takes precedence over `IP_GEOLOCATION_DB` and makes Plausible download (and keep up to date) a free MaxMind GeoLite2 MMDB of the selected edition (see below).

<sub><kbd>plausible-conf.env</kbd></sub>
```env
MAXMIND_LICENSE_KEY=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
```

---

#### `MAXMIND_EDITION`

Default: `GeoLite2-City`

### Email

#### `MAILER_ADAPTER`

Default: `Bamboo.SMTPAdapter`

The adapter to use for sending the mail.

---

#### `MAILER_EMAIL`

Default: `hello@plausible.local`

---

#### `MAILER_NAME`

If set, for example, to `Hello Plausible`, the mail would be sent with `from` combining `MAILER_NAME` and `MAILER_NAME` like this:

```
From: Hello Plausible <hello@plausible.local>
```

---

#### `POSTMARK_API_KEY`

---

#### `MAILGUN_API_KEY`

---

#### `MAILGUN_DOMAIN`

---

#### `MAILGUN_BASE_URI`

---

#### `MANDRILL_API_KEY`

---

#### `SENDGRID_API_KEY`

---

#### `SMTP_HOST_ADDR`

---

#### `SMTP_HOST_PORT`

---

#### `SMTP_USER_NAME`

---

#### `SMTP_USER_PWD`

---

#### `SMTP_HOST_SSL_ENABLED`

---

#### `SMTP_RETRIES`

---

#### `SMTP_MX_LOOKUPS_ENABLED`

### Misc

#### `LOG_FORMAT`

Default: `standard`

Configures the format that log line are printed in, supports `standard` and `json`

---

#### `STORAGE_DIR`

---

#### `SENTRY_DSN`

---

#### `LOG_FAILED_LOGIN_ATTEMPTS`

Default: `false`

---

#### `SECURE_COOKIE`

Default: `false`
