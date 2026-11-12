# Sharqey, self-hosted

Run the Sharqey server on your own infrastructure. Sharqey is an end-to-end encrypted password
and secrets manager: all encryption happens in the browser extension, the apps and the CLI, so
the server only ever stores encrypted data.

Images: `ghcr.io/sharqey/server:<version>` for `linux/amd64` and `linux/arm64`.

Hosted version: [app.sharqey.com](https://app.sharqey.com). More at [sharqey.com](https://sharqey.com).

## Requirements

- Docker with Compose
- A domain and a reverse proxy with TLS (Caddy, Traefik, nginx). Clients only connect over HTTPS.

## Install

```sh
curl -fsSLO https://raw.githubusercontent.com/sharqey/self-hosted/main/docker-compose.yml
curl -fsSL https://raw.githubusercontent.com/sharqey/self-hosted/main/.env.example -o .env
# edit .env: SHARQEY_VERSION, BASE_URL, POSTGRES_PASSWORD, JWT_SECRET
docker compose up -d
```

Point your reverse proxy at `127.0.0.1:8080`. With Caddy:

```
sharqey.example.com {
	reverse_proxy 127.0.0.1:8080
}
```

## First admin

On the first start with an empty database the server creates an instance admin with the login
`admin` and prints a one-time password to its log:

```sh
docker compose logs app | grep bootstrap
```

Sign in at `BASE_URL`, change the password, add a passkey or two-factor authentication, then
create accounts for your team in the admin panel (**Settings → Open admin panel**).

## Clients

- Browser extension: [Chrome Web Store](https://chromewebstore.google.com/detail/sharqey-password-manager/hkiigjjakecaebakedhhlhfaobfpjmge).
  On the sign-in screen choose **Self-hosted server** and enter your address.
- Desktop (macOS): [sharqey/desktop releases](https://github.com/sharqey/desktop/releases) or
  `brew install --cask sharqey/tap/sharqey`.
- CLI: `curl -fsSL https://sharqey.example.com/install.sh | sh` installs the latest release
  and points it at your server. Or `brew install sharqey/tap/sharqey`, or a binary from
  [sharqey/cli releases](https://github.com/sharqey/cli/releases), then set
  `SHARQEY_SERVER_URL`. Without access to GitHub, mirror the release files and set
  `CLI_DOWNLOAD_URL` on the server to that mirror.

## Single sign-on and the license

Everything works without a license, with no limits on users or organizations. Single sign-on
(Google, Apple, OIDC, SAML) needs a license in `SHARQEY_LICENSE`. A license covers every version
built before its expiry date, and the version you run keeps working after it expires.

## Updating

Change `SHARQEY_VERSION` in `.env` and run `docker compose up -d`. Database migrations run on
start. Pin a version instead of `latest`: with a license, an automatic updater could pull a
version built after your license expired, and that version won't start while SSO is configured.

To roll back, set the previous version and run `docker compose up -d` again.

## Backups

Everything is in PostgreSQL, including file secrets. Back up the `db` volume, for example with
`docker compose exec db pg_dump -U sharqey sharqey > sharqey.sql`, and test restoring it.
