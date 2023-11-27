Plausible is configured with environment variables, by default supplied via `plausible-conf.env` envfile. They are read at startup in [`config/runtime.exs`](https://github.com/plausible/analytics/blob/master/config/runtime.exs)

In the [container image](https://hub.docker.com/r/plausible/analytics) this script is located at `/app/releases/0.0.1/runtime.exs` and you can mount a custom one if, for example, you want to configure some option which doesn't have an environment varible in the default script.

Here're the currently supported ENV vars:

#### `LOG_FORMAT`
Configures the format that log line are printed in, supports `standard` and `json`, defaults to `standard`

---

#### `LISTEN_IP`

Configures the IP address to bind the listen socket for the web server, defaults to `0.0.0.0`. Note that setting it to `127.0.0.1` in a container would make the web server unavailable from the outside.

---

#### `PORT`

Configures the port to bind the listen socket for the web server, defaults to `8000`

---

#### `BASE_URL`

Configures the URL to use in link generation, doesn't have any defaults and needs to be provided in the ENV vars

```sh
BASE_URL=https://who.copycat.fun
```

---

#### `SECRET_KEY_BASE`

Configures the secret used for sessions in the dashboard, doesn't have any defaults and needs to be provided in the ENV vars, can be generated with `openssl rand -hex 64`


```sh
SECRET_KEY_BASE=NpS5g1bzwKMvMzqPXU9bGIW8U4qUYVWWxToWV6yLFtKQYvQkX7xsVv2aWckgvaKo
```

---

### `DATABASE_URL`

etc.

