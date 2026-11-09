# OpenQR — private LAN deployment on Portainer

## Settings already applied to this fork

- External port **18888**, internal application port **3000**.
- Image `openqr-lan:next16.4.5` is **built locally from this GitHub fork**,
  not pulled from the original publisher's prebuilt image.
- Exact patched Next.js **16.4.5** in `package.json`.
- Docker build uses `pnpm install --no-frozen-lockfile` because the original
  upstream lockfile still pins Next.js 16.2.9; the build validates that the
  installed version is **exactly 16.4.5**. For a fully reproducible build,
  regenerate/commit `pnpm-lock.yaml` with pnpm 10.16.1 when npm access is available.
- No database, account system, or built-in tracking.
- Container runs as UID 1001, read-only filesystem, all Linux capabilities
  dropped and `no-new-privileges` enabled.
- No Cloudflare Tunnel or router forwarding is configured by this stack.
- **This is an unauthenticated LAN service**: anyone who can reach TCP port
  18888 on your host can use it. Limit access with your network firewall/VLAN.
- Only when the user chooses the Location feature can the browser contact
  CARTO tile servers or OpenStreetMap Nominatim. QR generation does not
  contact these services.

## Portainer (local Docker socket)

1. Stacks → Add stack → `openqr` → Repository.
2. Repository URL: `https://github.com/hoangthang0106/openqr`.
3. Repository reference: `refs/heads/main`.
4. Compose path: `docker-compose.yml`.
5. Deploy stack. The first deployment must build the image from source.

Open `http://<CURRENT_DOCKER_HOST_LAN_IP>:18888` from your LAN.
Use the same host IP as Portainer, but change the port to 18888.

### If port 18888 is occupied
Check `docker ps --format "table {{.Names}}\t{{.Ports}}"` or
`sudo ss -lntp | grep ':18888'` on the Docker host before deploying.
Edit only the **host-side** port in the Compose file, e.g.
`"18889:3000"`.

### Portainer build limitation
Some Portainer installations connected via a remote **Agent** cannot build
`build:` services directly from Git. In that case build the Docker image
once on the Docker host using `docker compose build`, then deploy the stack
from Portainer with the local image. Do not fall back to the old official
image if security updates are important.

### LAN security
Do not configure Cloudflare Tunnel or router port forwarding to 18888.
Do not assume that `ports: "18888:3000"` enforces private access —
it binds host interfaces. Use firewall/VLAN rules to restrict clients.
