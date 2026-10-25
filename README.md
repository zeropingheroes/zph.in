# zph.in

YOURLS URL shortener for https://zph.in/, running on the official `yourls` Apache image. The root URL
redirects to https://zeropingheroes.co.uk/ (see `vhost.conf`).

## Files

| File | Purpose |
|---|---|
| `vhost.conf` | Apache vhost: YOURLS rewrites, plus the root redirect |
| `php-security.ini` | PHP hardening, mounted last into `conf.d` |
| `www/` | YOURLS install (gitignored). `user/` holds config and plugins |
| `backup-yourls*.sh` | Database dump plus `www/`, rsynced off-site, monitored by healthchecks.io |

## Updating PHP

The official image is rebuilt upstream whenever PHP ships a release. Pull it, recreate the container,
and check the version:

```bash
docker compose pull yourls-apache && docker compose up -d yourls-apache
docker compose exec yourls-apache php -v
```

## Updating YOURLS

The image only copies YOURLS into `/var/www/html` when `yourls-loader.php` is **missing**. Because
`www/` is a persistent bind mount, a new image tag updates PHP but **not** YOURLS itself. To upgrade
the application:

1. Back up: run the backup script, or at least `tar czf www-backup.tgz www`.
2. Bump the image tag (e.g. `yourls:1.10.6-apache`) in compose.
3. Move the old install aside, keeping only `user/`:
   ```bash
   mv www www.old && mkdir www && cp -a www.old/user www/ && chown 1000:988 www
   ```
4. `docker compose up -d`. The entrypoint copies the new release into `www/`. It keeps your
   `user/config.php` (because it isn't empty) and any plugin directories that already exist.
5. Copy over any other custom files from `www.old/` (favicon, robots.txt, etc.) if you had them.
6. Open `https://zph.in/admin/`. If the database schema changed, YOURLS shows an upgrade prompt.
   Run it.
7. Once everything works, remove `www.old`.
