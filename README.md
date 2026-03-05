# Nick Stack (Self-Hosted)

This repo bootstraps the self-hosted stack discussed for Nick:

- Immich (photos/videos)
- Syncthing (file sync)
- Forgejo (git forge)
- Headscale (self-hosted Tailscale control plane)
- Navidrome (Subsonic-compatible music server)

## Network Model

- `edge` network: app entry services (Immich, Syncthing, Forgejo, Navidrome)
- `backend` network (internal): database/cache only (Immich Postgres/Redis, Forgejo Postgres, Immich ML)
- `overlay-hardened` network (internal): hardened tailnet access path

Only edge services publish host ports. DB/Redis services stay isolated on internal Docker networking.

## Quick Start

1. Create env file:

```bash
cp .env.example .env
```

2. Update passwords, ports, and mount paths in `.env`.

3. Start the stack:

```bash
docker compose up -d
```

Optional profiles:

- Direct overlay client (tailnet can reach edge network directly):

```bash
docker compose --profile overlay up -d
```

- Hardened overlay (tailnet reaches only `edge-gateway`, which proxies edge services):

```bash
docker compose --profile hardened-overlay up -d
```

- Local Headscale control plane (optional dev mode, if not using DO droplet):

```bash
docker compose --profile local-headscale up -d
```

4. Open services:

- Immich: `http://localhost:${IMMICH_PORT}`
- Syncthing: `http://localhost:${SYNCTHING_UI_PORT}`
- Forgejo: `http://localhost:${FORGEJO_HTTP_PORT}`
- Headscale API: `http://localhost:${HEADSCALE_PORT}`
- Navidrome: `http://localhost:${NAVIDROME_PORT}`

## Headscale Bootstrap

If using your own Headscale server (for example on a DO droplet), set `TS_LOGIN_SERVER` and `TS_AUTHKEY` in `.env`.

If using the local Headscale profile, create a user and pre-auth key:

```bash
docker compose exec headscale headscale users create nick
docker compose exec headscale headscale preauthkeys create --user nick --reusable --expiration 24h
```

Use the generated key when joining a client.

## Data Layout

Persistent volumes are stored in `./data`:

- `data/immich/*`
- `data/syncthing/*`
- `data/forgejo*`
- `data/headscale`
- `data/navidrome/*`
- `data/tailscale/*`

## Notes

- `config/headscale/config.yaml` is intentionally minimal for local bootstrap.
- For remote client use, front Headscale with HTTPS and set `server_url` to your public URL.
- Navidrome is included as the Subsonic-compatible default. If you want Gonic instead, we can swap it in.
- Service image tags are pinned in `.env` to stable/LTS lines (update there when rolling forward).
