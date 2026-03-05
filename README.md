# Nick Stack (Self-Hosted)

This repo bootstraps the self-hosted stack discussed for Nick:

- Immich (photos/videos)
- Syncthing (file sync)
- Forgejo (git forge)
- Headscale (self-hosted Tailscale control plane)
- Navidrome (Subsonic-compatible music server)

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

4. Open services:

- Immich: `http://localhost:${IMMICH_PORT}`
- Syncthing: `http://localhost:${SYNCTHING_UI_PORT}`
- Forgejo: `http://localhost:${FORGEJO_HTTP_PORT}`
- Headscale API: `http://localhost:${HEADSCALE_PORT}`
- Navidrome: `http://localhost:${NAVIDROME_PORT}`

## Headscale Bootstrap

Create a user and pre-auth key:

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

## Notes

- `config/headscale/config.yaml` is intentionally minimal for local bootstrap.
- For real remote client use, front Headscale with HTTPS and set `server_url` to your public URL.
- Navidrome is included as the Subsonic-compatible default. If you want Gonic instead, we can swap it in.
