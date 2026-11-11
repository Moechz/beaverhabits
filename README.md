# beaverhabits-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [beaverhabits](https://github.com/daya0576/beaverhabits).

## What this is

A four-file TOS Docker application archive (`beaverhabitsdocker.tar.gz`) that installs
beaverhabits on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id `beaverhabitsdocker`)
- `beaverhabitsdocker.lang` — 23-language superset (TOS requires 14)
- `beaverhabitsdocker.svg` — application icon (908 bytes, 3 SVG elements)
- `docker-compose.yml` — one service, image `daya0576/beaverhabits:0.9.1`

## How it works

| | |
|---|---|
| Container image | `daya0576/beaverhabits:0.9.1` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8064/` (container port 8080) |
| Runs as | non-root, `user: "65534:65534"` |
| Persistent data | `/app/.user` → `/Volume1/DockerAppData/beaverhabitsdocker/user` |
| Logs | `docker logs beaverhabitsdocker` (stdout/stderr) |



## Design highlights

- **Non-root.** `user: "65534:65534"`; verified on real hardware (the container reports uid 65534).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no `network_mode: host`, and no Docker socket, no `cap_add`.
- **Health check** probing the service on container port 8080.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip without a timestamp, plus the review-standards self-check above.

## Runtime file manifest

Everything the application writes on the NAS lives inside its own data
directory; the container-internal `/tmp` is discarded with the container. The
table lists what is created and how it behaves. Nothing else is written.

| Path | Purpose | Created when | Growth bound | Lifecycle |
|---|---|---|---|---|
| `/Volume1/DockerAppData/beaverhabitsdocker/user` | beaverhabits application data (everything the container writes outside /tmp) | on first start / on demand | bounded by your own data | persistent; kept on uninstall unless you ask to delete it |

## Build

```bash
scripts/build.sh                  # -> out/beaverhabitsdocker.tar.gz + .sha256
scripts/build.sh 0.9.1-2   # packaging iteration bump
scripts/build.sh 0.9.1-1 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs
`tools/verify.py`, a review-standards self-check (archive layout, JSON validity,
required fields, 14+ language coverage, version consistency, Docker-Hub-only
images, fixed tag, reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/beaverhabitsdocker.tar.gz   # official spec check
bash test/install-and-verify.sh                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission checklist

1. Public GitHub repo (code + README only).
2. Release **tag `v0.9.1-1`**, assets `beaverhabitsdocker.tar.gz` +
   `beaverhabitsdocker.tar.gz.sha256` (Docker asset naming: `<app_id>.tar.gz`, **no
   platform suffix**).
3. Developer platform → Add Application: ID `beaverhabitsdocker`, package type Docker,
   repository URL, architecture `x86_64`.
4. Version Management → Add Version `0.9.1-1` → automated validation → review.

## License / branding

beaverhabits is distributed by its upstream authors under **BSD-3-Clause**. The container image is the official image from Docker Hub (`daya0576/beaverhabits:0.9.1`) and is used unmodified; no upstream source code is redistributed here. This packaging is an independent community submission by Moechz and is not affiliated with the upstream project or TerraMaster.

## Limitations

- The application runs inside a container; data lives in the application data directory (`/Volume1/DockerAppData/beaverhabitsdocker/`) as described above.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
