<p align="center">
    <img src="./images/plausible_logo.compressed.png">
</p>

<p align="center">
    <strong>A getting started guide to self-hosting Plausible Community Edition</strong>
</p>

**Contact**:

- For release announcements please go to [GitHub releases.](https://github.com/plausible/analytics/releases)

- For a question or advice please go to [GitHub discussions.](https://github.com/plausible/analytics/discussions/categories/self-hosted-support)

---

<p align="center">
    <a href="#installation">Installation</a> &bull;
    <a href="#configuration">Configuration</a> &bull;
    <a href="#reverse-proxy">Reverse Proxy</a> &bull;
    <a href="#google-search">Google Search</a> &bull;
    <a href="#faq">FAQ</a>
</p>

---

## Installation

Plausible Community Edition is designed to be self-hosted through Docker. You don't have to be a Docker expert to launch your own instance, but you should have a basic understanding of the command-line and networking to successfully set it up.

### Requirements

The only thing you need to install Plausible is a server with Docker installed. The server must have a CPU with x86_64 or arm64 architecture and support for SSE 4.2 or equivalent NEON instructions. We recommend using a minimum of 4GB of RAM but the requirements will depend on your site traffic. 

We've tested this on [Digital Ocean](https://m.do.co/c/91569eca0213) (affiliate link) but any hosting provider works. If your server doesn't come with Docker pre-installed, you can follow [their docs](https://docs.docker.com/get-docker/) to install it.

To make your Plausible instance accessible on a (sub)domain, you also need to be able to edit your DNS. Plausible isn't currently designed for subfolder installations.

### Quick start

To get started quickly, download the [plausible/hosting](https://github.com/plausible/hosting) repo as a starting point. It has everything you need to boot up your own Plausible server.

<sub><kbd>console</kbd></sub>
```console
$ git clone https://github.com/plausible/hosting
$ cd hosting
```

Alternatively, you can download and extract the repo as a tarball

<sub><kbd>console</kbd></sub>
```console
$ curl -L https://github.com/plausible/hosting/archive/master.tar.gz | tar -xz
$ cd hosting-master
```

In the downloaded directory you'll find two important files:

- [`docker-compose.yml`](https://github.com/plausible/hosting/blob/master/docker-compose.yml) - installs and orchestrates networking between your Plausible server, Postgres database, Clickhouse database (for stats), and an SMTP server. It comes with sensible defaults that are ready to go, although you're free to tweak the settings if you wish.
- [`plausible-conf.env`](https://github.com/plausible/hosting/blob/master/plausible-conf.env) - configures the Plausible server itself. Full configuration options are documented [below.](#configuration)

We'll need to populate the latter one with the required configuration. Right now it looks like this:

<sub><kbd>[plausible-conf.env](https://github.com/plausible/hosting/blob/master/plausible-conf.env)</kbd></sub>
```env
BASE_URL=replace-me
SECRET_KEY_BASE=replace-me
```

Let's replace these required ENV vars with out own values.

First we'll generate the `SECRET_KEY_BASE` using `openssl`

<sub><kbd>console</kbd></sub>
```console
$ openssl rand -base64 48
GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

And then we need to decide the URL where the instance would be accessible. Let's assume we decide to put it at `plausible.example.com`

<sub><kbd>plausible-conf.env</kbd></sub>
```env
BASE_URL=http://plausible.example.com
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
``````

Note that by default Plausible is not using HTTPS. We'll add a reverse proxy to fix it. An easy one to add is Caddy.

### Version management

Plausible follows [semantic versioning:](https://semver.org/) `MAJOR.MINOR.PATCH`

You can find available Plausible versions on [DockerHub](https://hub.docker.com/r/plausible/analytics). The default `latest` tag refers to the latest stable release tag. You can also pin your version:

* `plausible/analytics:v2` pins the major version to `2` but allows minor and patch version upgrades
* `plausible/analytics:v2.0` pins the minor version to `2.0` but allows only patch upgrades

None of the functionality is backported to older versions. If you wish to get the latest bug fixes and security updates you need to upgrade to a newer version.

New versions are published on [the releases page](https://github.com/plausible/analytics/releases) and their changes are documented in our [Changelog.](https://github.com/plausible/analytics/blob/master/CHANGELOG.md) Please note that database schema changes require running migrations when you're upgrading. However, we consider the schema
as an internal API and therefore schema changes aren't considered a breaking change.

## Configuration

Plausible is configured with environment variables, by default supplied via [<kbd>plausible-conf.env</kbd>](https://github.com/plausible/hosting/blob/master/plausible-conf.env) [env_file.](https://github.com/plausible/hosting/blob/bb6decee4d33ccf84eb235b6053443a01498db53/docker-compose.yml#L38-L39) They are read at startup in [`config/runtime.exs`](https://github.com/plausible/analytics/blob/master/config/runtime.exs)

In the [container image](https://hub.docker.com/r/plausible/analytics) this script is located at `/app/releases/0.0.1/runtime.exs` and you can mount a custom one if, for example, you want to configure some option which doesn't have an environment varible in the default script.

> Note that if you start a container with one set of ENV vars and then update the ENV vars and restart the container, they won't take effect due to the immutable nature of the containers. The container needs to be recreated.

Here's the minimal <kbd>plausible-conf.env</kbd>

```env
BASE_URL=https://example.com
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
```

And here's <kbd>plausible-conf.env</kbd> with extra configuration

```env
BASE_URL=https://example.com
SECRET_KEY_BASE=GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz
PORT=8000
MAXMIND_LICENSE_KEY=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
MAXMIND_EDITION=GeoLite2-City
GOOGLE_CLIENT_ID=140927866833-002gqg48rl4iku76lbkk0qhu0i0m7bia.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=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
MAILER_NAME=Plausible
MAILER_EMAIL=plausible@example.com
SENTRY_DSN=https://7f16d5d6ee70465789e082bd09481556@o1012425.ingest.sentry.io/6643873
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

### Registration

#### `DISABLE_REGISTRATION`

Default: `true`

Restricts registration of new users. Possible values are `true` (full restriction), `false` (no restriction), and `invite_only` (only the invited users can register).

---

#### `ENABLE_EMAIL_VERIFICATION`

Default: `false`

When enabled, new users need to verify their email addressby following a link delivered to their mailbox.

### Web

#### `LISTEN_IP`

Default: `0.0.0.0`

Configures the IP address to bind the listen socket for the web server.

> Note that setting it to `127.0.0.1` in a container would make the web server unavailable from outside the container.

---

#### `PORT`

Default: `8000`

Configures the port to bind the listen socket for the web server.

### Database

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

### Google

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

### Locations

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

### Email

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

## Google Search Integration

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

## FAQ

- useful clickhouse commands
- useful docker commands
- useful postgres commands
- useful plausible commands
