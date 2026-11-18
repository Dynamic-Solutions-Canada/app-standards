# Containers, reverse proxy and server operation

How a web product's production containers are defined, built, run and updated, and how the shared nginx reverse proxy in front of them is laid out. It also covers TLS, deploys, backups and health. Development containers belong to devkit (see `devkit/README.md`). The single-image shape is DIST-12 in [distribution.md](distribution.md), compose file names are CONF-11 in [settings-config.md](settings-config.md), secrets and non-root users are SEC-5 and SEC-7 in [security.md](security.md), log caps are LOG-4 in [logging.md](logging.md), and the deploy script's refusals are SCRIPT-10 in [dev-scripts.md](dev-scripts.md). This file links to those rules rather than restating them.

Counts are against the four web monorepos (persistent, printstream, game-is-up, store-sign-controller) plus phonelock's hub, the server component of an Android product, unless a rule says otherwise. PrintStream is reviewed last; where it diverges, its rule is noted as deferred.

## Overview

| Layer | Rule | Reference |
| --- | --- | --- |
| Image | CTR-1 to CTR-3 | `store-sign-controller/Dockerfile`; `printstream/Dockerfile` for the entrypoint |
| Compose | CTR-4 to CTR-7 | `persistent/compose.server.yml` for shape; `phonelock/server/compose.server.yml` for limits and hardening |
| Reverse proxy | CTR-8, CTR-9 | `phonelock/server/nginx.conf` and `nginx.proxy` |
| TLS | CTR-10 | `store-sign-controller/scripts/deploy/renew-certificate.sh` |
| Deploy, backup, health | CTR-11 to CTR-13 | `printstream` deploy script (backup before migrate); `phonelock/tools/deploy-hub.sh` (refusals and proxy install) |

## Rules

### CTR-1: The production image

- **Status:** Decided (2026-11-18). Existing in part (exact Node tag in 5 of 5; `npm ci` in 4 of 5 builds; runtime copies only built output in 1 of 5: store-sign-controller; `exec` to node in 3 of 5)
- **Applies to:** Web
- **Reference:** `store-sign-controller/Dockerfile` (explicit copies, `npm prune --omit=dev`, `exec`); `printstream/Dockerfile` (an entrypoint script that migrates, then `exec`s node)
- **Why:** An image that carries sources and dev dependencies is larger and has more to patch; a start command that is not `exec` leaves node unable to see SIGTERM, so every stop waits for the kill timeout.

- The base is `node:<version>-<distro>-slim` with the exact fleet version from PLAT-8 in [platform.md](platform.md), and a comment above the `FROM` names `.nvmrc` as the source of the version.
- Stages are build, then runtime. The build stage copies the package manifests and lockfile first, runs `npm ci`, then copies the sources.
- The runtime stage copies only what runs: built output, production dependencies (`npm prune --omit=dev` or a separate `npm ci --omit=dev`), migrations and the start script. It does not `COPY . .` from the build context.
- Every `apt-get install` ends with `rm -rf /var/lib/apt/lists/*` in the same layer.
- The start command makes node the main process: an entrypoint script that ends in `exec node ...`, or `sh -c "npm run db:migrate:deploy && exec node ..."`. Migrations run on start (DIST-12).
- The image carries the OCI labels `org.opencontainers.image.source`, `.revision` and `.version`, set from build arguments.
- The runtime user is SEC-7's.

### CTR-2: The build context

- **Status:** Decided (2026-11-18). Existing (the full secret list in 1 of 4: persistent)
- **Applies to:** Web
- **Reference:** `persistent/.dockerignore`
- **Why:** Anything in the context can reach an image layer, and stale build output in the context makes a build depend on the machine it ran on.

The root `.dockerignore` carries SEC-7's secret patterns at every depth, and also excludes `.git`, `**/node_modules`, `**/dist` and `**/*.tsbuildinfo`, so every build starts from sources. Each block has a one-line comment saying why.

### CTR-3: Third-party images are pinned and watched

- **Status:** Decided (2026-11-18). Existing (exact tags for first-party bases in 5 of 5; third-party images on the server mostly float)
- **Applies to:** Web, and every container on the shared server
- **Reference:** `devkit/infra/` (pins with a comment where a floating tag is deliberate)
- **Why:** A floating tag (`latest`, a bare major) changes what runs without any commit, and an image whose tag moved can no longer be rebuilt or compared.

Every image in a tracked compose file, and every container on the server, names an exact version tag (`postgres:16.6-bookworm`, `nginx:1.29.8`). A floating tag is allowed only with a comment naming why. Dependabot's `docker` ecosystem watches each repo's Dockerfiles and compose files, with Node's major held by an ignore rule because PLAT-8 moves it.

### CTR-4: Project and service names

- **Status:** Decided (2026-11-18). Existing (an explicit project `name:` in 1 of 5: store-sign-controller)
- **Applies to:** Web
- **Reference:** `store-sign-controller/compose.server.yml` (`name: store-sign`)
- **Why:** Without a fixed project name the folder name decides network, volume and container names, and the proxy's upstream names drift with it.

- Every compose file sets a top-level `name:`: the product's short name for production, with `-staging` or `-dev` for the others.
- Services use role names: `api`, `db`, `web` (only where the web app is a separate container), and a worker's own role name.
- `container_name` is not set. The service the proxy reaches gets a network alias on `nginx_default` (CTR-6) of `<product>-<service>`, and the proxy targets that alias.

### CTR-5: The service baseline

- **Status:** Decided (2026-11-18). Existing in part (restart policy, health checks on the database and `depends_on: service_healthy` in 5 of 5; `start_period` in 2 of 5; `init` in 1 of 5)
- **Applies to:** Web
- **Reference:** `persistent/compose.server.yml`
- **Why:** A service that restarts, reports health and starts in dependency order needs no hand-holding after a reboot or a deploy.

Every long-running service sets:

- `restart: unless-stopped`.
- A health check. The database uses `pg_isready`; the API calls its liveness route (PLAT-15) with node's own `fetch`, since slim images have no curl; every check sets `start_period` long enough to cover migrations.
- `depends_on` with `condition: service_healthy` for each service it needs.
- `init: true` on node services, so signals and zombie processes are handled.
- The shared logging anchor (LOG-4).
- An explicit `environment:` list, with required secrets as `${VAR:?message}` (SEC-5), and secret files mounted read only from `.secrets/` to `/run/secrets/` (SEC-7 owns their ownership and mode).
- Named volumes for data. Postgres 18 and later mounts its volume at `/var/lib/postgresql`, not `/var/lib/postgresql/data`.

### CTR-6: Networks and ports

- **Status:** Decided (2026-11-18). Existing (the external `nginx_default` network and no published ports in 3 of 3 tracked production files: persistent, store-sign-controller, phonelock)
- **Applies to:** Web, and every container on the shared server
- **Reference:** `persistent/compose.server.yml`
- **Why:** A published port bypasses the proxy, its TLS and its access log, and a database on the proxy's network is reachable from every other product on the server.

- Only the service the proxy reaches joins the external `nginx_default` network, with the CTR-4 alias. The database and workers stay on the project's own network.
- A worker that needs no outbound access sits on an `internal: true` network.
- No production service publishes a host port (SEC-7). An operator tool that must be reachable from the host binds `127.0.0.1:` explicitly, never a bare port or `0.0.0.0`.

### CTR-7: Resource limits and hardening

- **Status:** Decided (2026-11-18). Existing (limits on 1 of 5 hub or API services: phonelock's hub; the full hardening set only in printstream's slicer example)
- **Applies to:** Web, and every container on the shared server
- **Reference:** `phonelock/server/compose.server.yml` (limits overridable from the env file); `printstream/compose.server.example.yml` slicer service (the hardened worker)
- **Why:** One runaway container should not take the server's memory, and a service that parses files from the internet should not be able to do anything else.

- Every service sets `mem_limit` with an equal `memswap_limit`, `pids_limit`, and `security_opt: [no-new-privileges:true]`. Values come from the env file with a default in the compose file.
- A service that processes untrusted input (uploaded files, third-party documents) also sets `read_only: true` with `tmpfs` for its scratch paths, `cap_drop: [ALL]` with only the capabilities it needs added back, and `cpus`, and it refuses requests without a service token shared with its caller.

### CTR-8: The shared reverse proxy

- **Status:** Decided (2026-11-18). Existing (one nginx container owning ports 80 and 443 on `nginx_default`, a per-site file per product, per-site access logs with host log rotation, and a catch-all site)
- **Applies to:** Web (the shared server)
- **Reference:** the server's proxy configuration, which is not yet tracked in any repo (D-116)
- **Why:** One proxy keeps TLS, logging and routing in one place, and a predictable layout lets any product add or change its site without touching the others.

- One nginx container owns ports 80 and 443 and the `nginx_default` network. Its image is pinned (CTR-3) and its log is capped (LOG-4).
- Each product has one site file, `conf.d/<product>.conf`, installed by the deploy from the repo's tracked `nginx.server.conf` (CONF-11). Backups of a site file live outside `conf.d/`.
- A `000-default.conf` catch-all answers names no site claims: on port 80 it redirects to the company site, and on 443 it refuses the handshake (`ssl_reject_handshake on`), so no product's certificate or page is served for a stranger's name.
- The base configuration sets `server_tokens off` and keeps access logs off globally; each site names its own access log under `/var/log/nginx/sites/`, rotated daily on the host and kept 14 days.
- A change is applied with `nginx -t` and then a graceful reload, never a restart, and an install script restores the previous file if the test fails.

### CTR-9: The per-site configuration

- **Status:** Decided (2026-11-18). Existing in part (variable upstream with Docker's resolver in every site; real client address from Cloudflare in 1 site: phonelock)
- **Applies to:** Web
- **Reference:** `phonelock/server/nginx.conf` with its included `nginx.proxy` snippet
- **Why:** Behind Cloudflare every request arrives from Cloudflare's addresses, so without these settings an app logs and rate-limits the wrong client, and believes every request is plain HTTP.

- The upstream is resolved at request time: `resolver 127.0.0.11 valid=30s;` and `set $upstream <product>-<service>;` before `proxy_pass http://$upstream:<port>;`, so restarting the app container never needs a proxy reload.
- Proxy headers come from one shared include, not a copy per location: `Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Host`, and `X-Forwarded-Proto https` for a site behind Cloudflare (never `$scheme`, which is always `http` there).
- The real client address is restored from Cloudflare with `real_ip_header CF-Connecting-IP` and `set_real_ip_from` for Cloudflare's published ranges, kept in one shared include and refreshed when Cloudflare changes them.
- WebSocket routes use a `map $http_upgrade $connection_upgrade` and long read timeouts on that location only.
- `client_max_body_size` matches the app's own upload limit, stated in both places.
- A site that exposes only an API and a web app may allow-list its paths and return 404 for the rest.

### CTR-10: TLS and certificates

- **Status:** Decided (2026-11-18). Existing (one origin certificate, renewed by a scheduled script; every other site relies on Cloudflare's edge certificate)
- **Applies to:** Web (the shared server)
- **Reference:** `store-sign-controller/scripts/deploy/renew-certificate.sh`
- **Why:** An expired certificate takes a site down with no warning, and the leg between Cloudflare and the server is only as private as its own encryption.

- How the Cloudflare-to-server leg is encrypted is decided in D-113.
- A site whose certificate the server holds renews it with a pinned `certbot` container using the webroot method, an ACME location in its site file (`location ^~ /.well-known/acme-challenge/`), a lock so runs never overlap, and a deploy hook that runs `nginx -t` and reloads. The script fails, and alerts (CTR-13), when fewer than 21 days remain.

### CTR-11: Deploys

- **Status:** Decided (2026-11-18). Existing in part (a scripted deploy in 4 of 5; a database backup before migrations in 1 of 5)
- **Applies to:** Web
- **Reference:** printstream's deploy script (backup before migrate, image pinning, prune; deferred while printstream is last); `phonelock/tools/deploy-hub.sh` (refusals, proxy install with rollback, health probe)
- **Why:** A deploy that migrates without a backup cannot be undone, and one that does not wait for health reports success for a broken release.

The deploy script (SCRIPT-10) runs these steps in order: refuse unless the tree is clean and on the release branch; dump the database to the product's backup folder and verify the dump; update the server checkout (or pull the release image, per D-115); `docker compose up -d`; wait for every service to report healthy, and stop with the logs if one does not; install the site file per CTR-8 if it changed; then prune dangling images. The server keeps no untracked compose file other than `compose.override.yml` for values specific to that host, and no `*.bak` copies in the deploy folder.

### CTR-12: Backups and restores

- **Status:** Decided (2026-11-18). Existing (the server is backed up whole at the hypervisor level; a pre-deploy dump in 1 of 5)
- **Applies to:** Web
- **Reference:** SEC-13 in [security.md](security.md) owns what each product promises its users
- **Why:** A whole-machine backup restores everything at once; restoring one product's data needs that product's own dump and a procedure someone has tried.

The server is backed up whole by its hypervisor, which covers every product's volumes. Each product also keeps the dumps CTR-11 takes before migrations (the newest ten), and its deployment document states how to restore one into a fresh database. That procedure is tried once on a disposable database and the date recorded.

### CTR-13: Health reporting

- **Status:** Decided (2026-11-18). Existing (health checks on most services; nothing reports an unhealthy container or a failed renewal)
- **Applies to:** Web (the shared server)
- **Reference:** none yet (D-114)
- **Why:** A health check that nobody sees only helps the restart policy; a failing certificate renewal or a container stuck unhealthy should reach the maintainer.

Every long-running container has a health check (CTR-5). A container that stays unhealthy, a certificate renewal that fails, and a deploy that stops are reported to the maintainer through the channel D-114 chooses.
