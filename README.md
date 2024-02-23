<p align="center">
    <picture>
        <source media="(prefers-color-scheme: dark)" srcset="./images/logo_dark.svg" width="300">
        <source media="(prefers-color-scheme: light)" srcset="./images/logo_light.svg" width="300">
        <img src="./images/logo_light.svg" width="300">
    </picture>
</p>

<p align="center">
    <strong>A getting started guide to self-hosting Plausible Community Edition</strong>
</p>

<!-- TODO latest version, current version, requirements -->

**Contact**:

- For release announcements please go to [GitHub releases.](https://github.com/plausible/analytics/releases)

- For a question or advice please go to [GitHub discussions.](https://github.com/plausible/analytics/discussions/categories/self-hosted-support)

---

<p align="center">
    <a href="#install">Install</a> &bull;
    <a href="#upgrade">Upgrade</a> &bull;
    <a href="#configure">Configure</a> &bull;
    <a href="#integrate">Integrate</a> &bull;
    <a href="#faq">FAQ</a>
</p>

---

## Install

Plausible Community Edition (or CE for short) is designed to be self-hosted through Docker. You don't have to be a Docker expert to launch your own instance, but you should have a basic understanding of the command-line and networking to successfully set it up.

### Requirements

The only thing you need to install Plausible CE is a server with Docker. The server must have a CPU with x86_64 or arm64 architecture and support for SSE 4.2 or equivalent NEON instructions. We recommend using a minimum of 4GB of RAM but the requirements will depend on your site traffic. 

We've tested this on [Digital Ocean](https://m.do.co/c/91569eca0213) (affiliate link) but any hosting provider works. If your server doesn't come with Docker pre-installed, you can follow [their docs](https://docs.docker.com/get-docker/) to install it.

To make your Plausible CE instance accessible on a (sub)domain, you also need to be able to edit your DNS. Plausible CE isn't currently designed for subfolder installations.

### Quick start

To get started quickly, clone the [plausible/hosting](https://github.com/plausible/hosting) repo. It has everything you need to boot up your own Plausible CE server.

<sub><kbd>console</kbd></sub>
```console
$ git clone https://github.com/plausible/hosting
$ cd hosting
```

In the downloaded directory you'll find two important files:

- [`docker-compose.yml`](https://github.com/plausible/hosting/blob/master/docker-compose.yml) - installs and orchestrates networking between your Plausible CE server, Postgres database, Clickhouse database (for stats), and an SMTP server.
- [`plausible-conf.env`](https://github.com/plausible/hosting/blob/master/plausible-conf.env) - configures the Plausible server itself. Full configuration options are documented [below.](#configuration)

Right now the latter looks like this:

<sub><kbd>[plausible-conf.env](https://github.com/plausible/hosting/blob/master/plausible-conf.env)</kbd></sub>
```env
BASE_URL=replace-me
SECRET_KEY_BASE=replace-me
```

Let's do as it asks and populate these required environment variables with our own values.

First we generate the `SECRET_KEY_BASE` using `openssl`

<sub><kbd>console</kbd></sub>
```console
$ openssl rand -base64 48
GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

And then we decide on the URL where the instance would be accessible. Let's assume we choose `plausible.example.com`

<sub><kbd>plausible-conf.env</kbd></sub>
```diff
- BASE_URL=replace-me
+ BASE_URL=http://plausible.example.com
- SECRET_KEY_BASE=replace-me
+ SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

We can start our instance now but the requests would be served over HTTP. Not cool! Let's configure [Caddy](https://caddyserver.com) to enable HTTPS.

<!-- TODO note about CloudFlare -->

> For other reverse-proxy setups please see [reverse-proxy](https://github.com/plausible/hosting/tree/master/reverse-proxy) docs.

First we need to point DNS records for `plausible.example.com` to the IP address of the instance. This is needed for Caddy to issue the TLS certificates.

Then we need to let Caddy know the domain name for which to issue the TLS certificate and the service to redirect the requests to.

<sub><kbd>[reverse-proxy/docker-compose.caddy-gen.yml](https://github.com/plausible/hosting/blob/master/reverse-proxy/docker-compose.caddy-gen.yml)</kbd></sub>
```diff
  plausible:
    labels:
-     virtual.host: "example.com" # change to your domain name
+     virtual.host: "plausible.example.com"
      virtual.port: "8000"
-     virtual.tls-email: "admin@example.com" # change to your email
+     virtual.tls-email: "admin@plausible.example.com"
```

Finally we need to update `BASE_URL` to use `https://` scheme.

<sub><kbd>plausible-conf.env</kbd></sub>
```diff
- BASE_URL=http://plausible.example.com
+ BASE_URL=https://plausible.example.com
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

Now we can start everything together.

<sub><kbd>console</kbd></sub>
```console
$ docker compose -f docker-compose.yml -f reverse-proxy/docker-compose.caddy-gen.yml up -d
```

It takes some time to start PostgreSQL and ClickHouse, create the databases, and run the migrations. After about fifteen seconds you should be able to access your instance at `BASE_URL`

In case something feels off, make sure to check out the logs

<sub><kbd>console</kbd></sub>
```console
$ docker compose logs -f
```

and start a [GitHub discussion.](https://github.com/plausible/analytics/discussions/categories/self-hosted-support)

Happy hosting!

Next we'll go over how to upgrade the instance when a new release comes out, more things to configure, and how to integrate with Google and others!

## Upgrade

Each new [release](https://github.com/plausible/analytics/releases/tag/v2.0.0) contains information on how to upgrade to it from the previous version. This section outlines the 
general steps and explains the versioning.

### Version management

Plausible CE follows [semantic versioning:](https://semver.org/) `MAJOR.MINOR.PATCH`

You can find available Plausible versions on [DockerHub](https://hub.docker.com/r/plausible/analytics). The default `latest` tag refers to the latest stable release tag. You can also pin your version:

- `plausible/analytics:v2` pins the major version to `2` but allows minor and patch version upgrades
- `plausible/analytics:v2.0` pins the minor version to `2.0` but allows only patch upgrades

None of the functionality is backported to older versions. If you wish to get the latest bug fixes and security updates you need to upgrade to a newer version.

New versions are published on [the releases page](https://github.com/plausible/analytics/releases) and their changes are documented in our [Changelog.](https://github.com/plausible/analytics/blob/master/CHANGELOG.md) Please note that database schema changes require running migrations when you're upgrading. However, we consider the schema
as an internal API and therefore schema changes aren't considered a breaking change.

We recommend to pin the major version instead of using `latest`. Either way the general flow for upgrading between minor version would look like this:

```console
$ cd hosting
$ docker compose down --remove-orphans
$ docker compose pull plausible
$ docker compose -f docker-compose.yml -f reverse-proxy/docker-compose.caddy-gen.yml up -d
```

> You can omit <kbd>-f docker-compose.yml -f reverse-proxy/docker-compose.caddy-gen.yml</kbd> if you are not using Caddy.

Changes in major versions would involve performing a data migration (e.g.[v2.0.0](https://github.com/plausible/analytics/releases/tag/v2.0.0)) or some other extra step.

## Configure

Plausible is configured with environment variables, by default supplied via [<kbd>plausible-conf.env</kbd>](https://github.com/plausible/hosting/blob/master/plausible-conf.env) [env_file.](https://github.com/plausible/hosting/blob/bb6decee4d33ccf84eb235b6053443a01498db53/docker-compose.yml#L38-L39) They are read at startup in [`config/runtime.exs`](https://github.com/plausible/analytics/blob/master/config/runtime.exs)

In the [container image](https://hub.docker.com/r/plausible/analytics) this script is located at `/app/releases/0.0.1/runtime.exs` and you can mount a custom one if, for example, you want to configure some option which doesn't have an environment varible in the default script.

> Note that if you start a container with one set of ENV vars and then update the ENV vars and restart the container, they won't take effect due to the immutable nature of the containers. The container needs to be recreated.

Here's the minimal <kbd>plausible-conf.env</kbd>

```env
BASE_URL=https://plausible.example.com
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

And here's <kbd>plausible-conf.env</kbd> with extra configuration

```env
BASE_URL=https://plausible.example.com
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
PORT=8000
MAXMIND_LICENSE_KEY=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
MAXMIND_EDITION=GeoLite2-City
GOOGLE_CLIENT_ID=140927866833-002gqg48rl4iku76lbkk0qhu0i0m7bia.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=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
MAILER_NAME=Plausible
MAILER_EMAIL=plausible@plausible.example.com
SENTRY_DSN=https://7f16e5d6dd72465789e081bd09481556@o1012425.ingest.sentry.io/6643873
DISABLE_REGISTRATION=invite_only
```

Here're the currently supported ENV vars:

### Required

#### `BASE_URL`

Configures the base URL to use in link generation, doesn't have any defaults and needs to be provided in the ENV vars

<sub><kbd>plausible-conf.env</kbd></sub>
```env
BASE_URL=https://example.fun
```

> In production systems, this should be your ingress host (CDN or proxy).

---

#### `SECRET_KEY_BASE`

Configures the secret used for sessions in the dashboard, doesn't have any defaults and needs to be provided in the ENV vars, can be generated with `openssl rand -base64 48`

<sub><kbd>console</kbd></sub>
```console
$ openssl rand -base64 48
GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

<sub><kbd>plausible-conf.env</kbd></sub>
```env
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
``````

> ⚠️ Don't use this exact value or someone would be able to sign a cookie with `user_id=1` and log in as the admin!

### Optional

<details>
<summary>Registration</summary>

#### `DISABLE_REGISTRATION`

Default: `true`

Restricts registration of new users. Possible values are `true` (full restriction), `false` (no restriction), and `invite_only` (only the invited users can register).

---

#### `ENABLE_EMAIL_VERIFICATION`

Default: `false`

When enabled, new users need to verify their email addressby following a link delivered to their mailbox.

</details>
<details>
<summary>Web</summary>

#### `LISTEN_IP`

Default: `0.0.0.0`

Configures the IP address to bind the listen socket for the web server.

> Note that setting it to `127.0.0.1` in a container would make the web server unavailable from outside the container.

---

#### `PORT`

Default: `8000`

Configures the port to bind the listen socket for the web server.

</details>
<details>
<summary>Database</summary>

Plausible uses PostgreSQL for storing user data and ClickhouseDB for analytics data. Use the following variables to configure them.

#### `DATABASE_URL`

Default: `postgres://postgres:postgres@plausible_db:5432/plausible_db`

Configures the URL for PostgreSQL database.

---

#### `CLICKHOUSE_DATABASE_URL`

Default: `http://plausible_events_db:8123/plausible_events_db`

Configures the URL for ClickHouse database.

---

#### `ECTO_IPV6`

Enables Ecto to use IPv6 when connecting to the PostgreSQL database. Not set by default.

<sub><kbd>plausible-conf.env</kbd></sub>
```env
ECTO_IPV6=true
```

---

#### `ECTO_CH_IPV6`

Enables Ecto to use IPv6 when connecting to the ClickHouse database. Not set by default.

<sub><kbd>plausible-conf.env</kbd></sub>
```env
ECTO_CH_IPV6=true
```

</details>
<details>
<summary>Google</summary>

For step-by-step integration with Google [see below.](#google-search-integration)

#### `GOOGLE_CLIENT_ID`

The Client ID from the Google API Console for your project. Not set by default.

<sub><kbd>plausible-conf.env</kbd></sub>
```env
GOOGLE_CLIENT_ID=140927866833-002gqg48rl4iku76lbkk0qhu0i0m7bia.apps.googleusercontent.com
```

---

#### `GOOGLE_CLIENT_SECRET`

The Client Secret from the Google API Console for your project. Not set by default.

<sub><kbd>plausible-conf.env</kbd></sub>
```env
GOOGLE_CLIENT_SECRET=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
```

</details>
<details>
<summary>Locations</summary>

#### `IP_GEOLOCATION_DB`

Default: `/app/lib/plausible-0.0.1/priv/geodb/dbip-country.mmdb.gz`

Defaults to the `.mmdb` file shipped within the container image.

This database is used to lookup GeoName IDs for IP addresses.

---

#### `GEONAMES_SOURCE_FILE`

Default: `/app/lib/location-0.1.0/priv/geonames.lite.csv`

Defaults to the one shipped within the container image.

This file is used to turn GeoName IDs into human readable strings for display on the dashboard.

---

#### `MAXMIND_LICENSE_KEY`

If set, this ENV variable takes precedence over `IP_GEOLOCATION_DB` and makes Plausible download (and keep up to date) a free MaxMind GeoLite2 MMDB of the selected edition (see below).

<sub><kbd>plausible-conf.env</kbd></sub>
```env
MAXMIND_LICENSE_KEY=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
```

---

#### `MAXMIND_EDITION`

MaxMind database edition to use (only if `MAXMIND_LICENSE_KEY` is set)

Default: `GeoLite2-City`

</details>
<details>
<summary>Email</summary>

Plausible CE uses a SMTP server to send transactional emails e.g. account activation, password reset. In addition, it sends non-transactional emails like weekly or monthly reports.

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

</details>
<details>
<summary>Misc</summary>

#### `SENTRY_DSN`

---

#### `LOG_FAILED_LOGIN_ATTEMPTS`

Default: `false`

---

#### `SECURE_COOKIE`

Default: `false`

</details>

## Integrate

<details>
<summary>Google</summary>

Integrating with Google either to get search keywords for hits from Google search or for imports from Universal Analytics can be frustrating.

The following screenshot-annotated guide shows how to do it all in an easy way: follow the Google-colored arrows.

Here's the outline of what we'll do:

- Set up OAuth on Google Cloud
    - Select or create a Google Cloud project
    - Register an OAuth application for a domain
    - Issue an OAuth client and key for that application
    - Verify the chosen domain on Google Search console
- Integrate with Google Search
    - Enable APIs for Google Search integration
    - Link it with Plausible
- Import historical data from Universal Analytics
    - Enable APIs for exports on Google Cloud
    - Import into Plausible

### Set up OAuth on Google Cloud

#### Select ot create a Google Cloud project

Go to [Google Cloud console,](http://console.cloud.google.com/) for example, by clicking <kbd>Go to console</kbd> on [Google Cloud landing page.](https://cloud.google.com) If Google asks you to register, just do it.

<img src="./images/0-google-cloud.png">

Once there, select a project that you want to use for Plausible OAuth app.

<img src="./images/1-project-select.png">

If you don't have a project already, or if you want to isolate Plausible from all your other Google Cloud things, you should create a new project.

---

<details><summary>Here's how to create a new Google Cloud project</summary>

In the <kbd>Select a project</kbd> pop-up, click <kbd>New project</kbd>

<img src="./images/1-project-new.png">

Pick a descriptive name. Organizations don't matter.

<img src="./images/1-project-create.png">

Once the project is created, select it and make sure all the other steps happen within that project. Google is tricky and sometimes switches you to a "default" project.

<img src="./images/1-project-created.png">

And just like that, you have a new Google Cloud project! Please do make sure it stays selected.

</details>

---

#### Register an OAuth application for a domain

Search for <kbd>APIs & Services</kbd> or something like that.

<img src="./images/2-app-registration-api-and-services-pick.png">

Then in the left sidebar pick <kbd>OAuth consent screen</kbd> to begin the OAuth application registration.

<img src="./images/2-app-registration-pick.png">

Choose <kbd>External</kbd> for the application type since the other one requires a Google Workspace and that costs money.

<img src="./images/2-app-registration-external.png">

On the next screen pick the name for your application and add your contact information.

<img src="./images/2-app-registration-consent-screen-0.png">

Scroll down -- skipping optional fields -- and type in your domain name and contact information (again).

<img src="./images/2-app-registration-consent-screen-1.png">

Skip the scopes.

<img src="./images/2-app-registration-scopes-skip.png">

Pick yourself as the test user, Google might complain about it but it works.

<img src="./images/2-app-registration-test-users.png">

Click the final <kbd>Save and continue</kbd> and you have the OAuth application registered.

#### Issue an OAuth client and key for that application

Pick <kbd>Credentials</kbd> in the sidebar.

<img src="./images/3-oauth-client-credentials-pick.png">

Click <kbd>+ Create credentials</kbd> dropdown and select <kbd>OAuth client ID</kbd>

<img src="./images/3-oauth-client-pick.png">

Pick <kbd>Web application</kbd> for the application type, type the name for the client, and add the redirect URL.

<img src="./images/3-oauth-client-create.png">

That redirect URL should be `/auth/google/callback` on your Plausible instance's [<kbd>BASE_URL</kbd>](https://github.com/plausible/hosting/blob/bb6decee4d33ccf84eb235b6053443a01498db53/plausible-conf.env#L1)

<img src="./images/3-oauth-client-created.png">

Copy these to your [<kbd>plausible-conf.env</kbd>](https://github.com/plausible/hosting/blob/master/plausible-conf.env) and make sure to recreate the `plausible` container since the ENV vars provided on startup get "baked in"

<sub><kbd>plausible-conf.env</kbd></sub>
```env
GOOGLE_CLIENT_ID=974728454958-e1vcqqqs6hmoc394663kjrkgfajrifdg.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=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
```
<sub><kbd>console</kbd></sub>
```console
$ docker compose stop plausible
[+] Running 1/1
 ⠿ Container hosting-plausible-1  Stopped
$ docker compose rm plausible
? Going to remove hosting-plausible-1 Yes
[+] Running 1/0
 ⠿ Container hosting-plausible-1  Removed
$ docker compose create plausible
[+] Running 4/0
 ⠿ Container hosting-plausible_events_db-1  Running 0.0s
 ⠿ Container hosting-mail-1                 Running 0.0s
 ⠿ Container hosting-plausible_db-1         Running 0.0s
 ⠿ Container hosting-plausible-1            Created
$ docker compose start plausible
[+] Running 3/3
 ⠿ Container hosting-plausible_db-1         Healthy 0.5s
 ⠿ Container hosting-plausible_events_db-1  Healthy 0.5s
 ⠿ Container hosting-plausible-1            Started
$ docker compose exec plausible sh -c 'echo $GOOGLE_CLIENT_ID'
974728454958-e1vcqqqs6hmoc394663kjrkgfajrifdg.apps.googleusercontent.com
```

#### Verify the chosen domain on Google Search console

Did you notice that during OAuth application registratation there was a note about Authorized URLs saying that they need to be verified? Nevermind, we are doing it now.

Start by navigating to [Google Search Console page.](http://search.google.com/u/1/search-console/welcome) 

Once there, either ensure that you've already verified your domain by checking the properties in the <kbd>Select property</kbd> dropdown on the left or pick one of the two ways to verify it. I only have screenshots for the "Domain" type so that's what I'm picking.

<img src="./images/4-search-console-new.png">

Whichever you pick, just follow the instruction in the pop-up, they are good.

<img src="./images/4-search-console-verify.png">

Success looks like this.

<img src="./images/4-search-console-verified.png">

With that, you are ready to integrate Plausible with Google Search and import Universal Analytics data. You can do both, neither, and anything in between.

### Integrate with Google Search

#### Enable APIs for Google Search integration

Go back to [Google Cloud,](https://console.cloud.google.com) ensure you have the correct project selected, and search for <kbd>Google Search Console API</kbd>

<img src="./images/5-search-console-api-search.png">

Enable it.

<img src="./images/5-search-console-api-enable.png">

#### Link it with Plausible

Go to the site settings on your Plausible dashboard.

<img src="./images/6-plausible-settings-pick.png">

In the settings select <kbd>Search Console</kbd> and press <kbd>Continue with Google</kbd>

> If you see a warning instead, that means you haven't set the <kbd>GOOGLE_CLIENT_ID</kbd> and <kbd>GOOGLE_CLIENT_SECRET</kbd> environment variables [correctly.](#issue-an-oauth-client-and-key-for-that-application)

<img src="./images/6-plausible-settings-search-console.png">

Choose the account that you added as the test user.

<img src="./images/6-choose-google-account.png">

Trust our own application.

<img src="./images/6-continue.png">

Allow viewing Search Console data.

<img src="./images/6-view-search-console-data.png">

Pick the property from Search Console.

<img src="./images/6-property.png">

And now we should be able to drilldown into Google search terms like on [plausible.io](https://plausible.io/plausible.io/referrers/Google?source=Google)

### Import historical data from Universal Analytics

#### Enable APIs for exports on Google Cloud

Go back to [Google Cloud,](https://console.cloud.google.com) ensure you have the correct project selected, and search for <kbd>Google Analytics API</kbd>

<img src="./images/7-analytics-api-search.png">

Enable it.

<img src="./images/7-analytics-api-enable.png">

Next search for <kbd>Google Analytics Reporting API</kbd>

<img src="./images/7-analytics-reporting-api-search.png">

And enable it.

<img src="./images/7-analytics-reporting-api-enable.png">

#### Import into Plausible

Go to the site settings on your Plausible dashboard.

<img src="./images/6-plausible-settings-pick.png">

In the <kbd>General</kbd> settings section scroll down to <kbd>Data Import from Google Analytics</kbd> and press  <kbd>Continue with Google</kbd> button.

> If you see a warning instead, that means you haven't set the <kbd>GOOGLE_CLIENT_ID</kbd> and <kbd>GOOGLE_CLIENT_SECRET</kbd> environment variables [correctly.](#issue-an-oauth-client-and-key-for-that-application)

<img src="./images/6-data-import.png">

Choose the account that you added as the test user.

<img src="./images/6-choose-google-account.png">

Trust our own application.

<img src="./images/6-continue.png">

Pick the view to import and then follow the Plausible directions.

<img src="./images/6-pick-view.png">

You'll receive an email once the data is imported.

</details>
<details>
<summary>MaxMind</summary>

</details>

## FAQ

<details>
<summary>How do I access Plausible from terminal?</summary>

You can starts an Interactive Elixir session from within the `plausible` container:

<sub><kbd>console</kbd></sub>
```console
$ cd hosting
$ docker compose exec plausible bin/plausible remote
```
```elixir
iex> Application.get_all_env :plausible
[
  {PlausibleWeb.Endpoint,
   [
     live_view: [signing_salt: "f+bZg/crMtgjZJJY7X6OwIWc3XJR2C5Y"],
     pubsub_server: Plausible.PubSub,
     render_errors: [
       view: PlausibleWeb.ErrorView,
       layout: {PlausibleWeb.LayoutView, "base_error.html"},
       accepts: ["html", "json"]
     ]
# etc.
# use ^C^C (ctrl+ctrl) to exit
```

</details>

<details>
<summary>How do I access ClickHouse from terminal?</summary>

You can starts a `clickhouse client` session from within the `plausible_events_db` container:

<sub><kbd>console</kbd></sub>
```console
$ cd hosting
$ docker compose exec plausible_events_db clickhouse client --database plausible_events_db
```
```sql
:) show tables

-- ┌─name───────────────────────┐
-- │ events                     │
-- │ events_v2                  │
-- │ imported_browsers          │
-- │ imported_devices           │
-- │ imported_entry_pages       │
-- │ imported_exit_pages        │
-- │ imported_locations         │
-- │ imported_operating_systems │
-- │ imported_pages             │
-- │ imported_sources           │
-- │ imported_visitors          │
-- │ ingest_counters            │
-- │ schema_migrations          │
-- │ sessions                   │
-- │ sessions_v2                │
-- └────────────────────────────┘

:) exit

-- Bye
```

</details>

<details>
<summary>How do I access PostgreSQL from terminal?</summary>

You can starts a `psql` session from within the `plausible_db` container:

<sub><kbd>console</kbd></sub>
```console
$ cd hosting
$ docker compose exec plausible_db psql -U postgres -h localhost -d plausible_db
```
```sql
plausible_db=# \d

--  Schema |                      Name                      |   Type   |  Owner
-- --------+------------------------------------------------+----------+----------
--  public | api_keys                                       | table    | postgres
--  public | api_keys_id_seq                                | sequence | postgres
--  public | check_stats_emails                             | table    | postgres
--  public | check_stats_emails_id_seq                      | sequence | postgres
--  public | create_site_emails                             | table    | postgres
--  public | create_site_emails_id_seq                      | sequence | postgres
--  public | email_activation_codes                         | table    | postgres
--  public | email_activation_codes_id_seq                  | sequence | postgres
--  public | email_verification_codes                       | table    | postgres
--  public | enterprise_plans                               | table    | postgres
--  public | enterprise_plans_id_seq                        | sequence | postgres
--  public | feedback_emails                                | table    | postgres
--  public | feedback_emails_id_seq                         | sequence | postgres
--  public | fun_with_flags_toggles                         | table    | postgres
--  public | fun_with_flags_toggles_id_seq                  | sequence | postgres
--  public | funnel_steps                                   | table    | postgres
--  public | funnel_steps_id_seq                            | sequence | postgres
--  public | funnels                                        | table    | postgres
--  public | funnels_id_seq                                 | sequence | postgres
--  public | goals                                          | table    | postgres
--  public | goals_id_seq                                   | sequence | postgres
--  public | google_auth                                    | table    | postgres
--  public | google_auth_id_seq                             | sequence | postgres
--  public | intro_emails                                   | table    | postgres
--  public | intro_emails_id_seq                            | sequence | postgres
--  public | invitations                                    | table    | postgres
--  public | invitations_id_seq                             | sequence | postgres
--  public | monthly_reports                                | table    | postgres
--  public | monthly_reports_id_seq                         | sequence | postgres
--  public | oban_jobs                                      | table    | postgres
--  public | oban_jobs_id_seq                               | sequence | postgres
--  public | oban_peers                                     | table    | postgres
--  public | plugins_api_tokens                             | table    | postgres
--  public | salts                                          | table    | postgres
--  public | salts_id_seq                                   | sequence | postgres
--  public | schema_migrations                              | table    | postgres
--  public | sent_accept_traffic_until_notifications        | table    | postgres
--  public | sent_accept_traffic_until_notifications_id_seq | sequence | postgres
--  public | sent_monthly_reports                           | table    | postgres
--  public | sent_monthly_reports_id_seq                    | sequence | postgres
--  public | sent_renewal_notifications                     | table    | postgres
--  public | sent_renewal_notifications_id_seq              | sequence | postgres
--  public | sent_weekly_reports                            | table    | postgres
--  public | sent_weekly_reports_id_seq                     | sequence | postgres
--  public | setup_help_emails                              | table    | postgres
--  public | setup_help_emails_id_seq                       | sequence | postgres
--  public | setup_success_emails                           | table    | postgres
--  public | setup_success_emails_id_seq                    | sequence | postgres
--  public | shared_links                                   | table    | postgres
--  public | shared_links_id_seq                            | sequence | postgres
--  public | shield_rules_ip                                | table    | postgres
--  public | site_imports                                   | table    | postgres
--  public | site_imports_id_seq                            | sequence | postgres
--  public | site_memberships                               | table    | postgres
--  public | site_memberships_id_seq                        | sequence | postgres
--  public | site_user_preferences                          | table    | postgres
--  public | site_user_preferences_id_seq                   | sequence | postgres
--  public | sites                                          | table    | postgres
--  public | sites_id_seq                                   | sequence | postgres
--  public | spike_notifications                            | table    | postgres
--  public | spike_notifications_id_seq                     | sequence | postgres
--  public | subscriptions                                  | table    | postgres
--  public | subscriptions_id_seq                           | sequence | postgres
--  public | totp_recovery_codes                            | table    | postgres
--  public | totp_recovery_codes_id_seq                     | sequence | postgres
--  public | users                                          | table    | postgres
--  public | users_id_seq                                   | sequence | postgres
--  public | weekly_reports                                 | table    | postgres
--  public | weekly_reports_id_seq                          | sequence | postgres

plausible_db=# exit
```

</details>
