# knot-helm-chart

A Helm chart that deploys a [knot](https://docs.tangled.org/knots) — the git server of
[Tangled](https://tangled.org), the code collaboration platform built on ATProto.

```bash
helm repo add knot https://qjoly.github.io/knot-helm-chart
helm install knot knot/knot \
  --set common.variables.nonSecret.KNOT_SERVER_HOSTNAME=knot.example.com \
  --set common.variables.nonSecret.KNOT_SERVER_OWNER=did:plc:xxxxxxxxxxxxxxxxxxxxxxxx \
  --set common.ingress.enabled=true \
  --set common.ingress.hostName=knot.example.com
```

Then hit **verify** on <https://tangled.org/settings/knots>. That is the whole registration: the
appview fetches `https://<hostname>/xrpc/sh.tangled.owner` and compares the DID it returns to yours.

## What you have to provide

| | |
|---|---|
| A public FQDN | It becomes the knot's identity, `did:web:<hostname>` |
| HTTPS on 443 for it | An Ingress to port 5555 |
| **SSH on port 22 for the same name** | Not optional — see below |
| Your ATProto DID | `curl "https://public.api.bsky.app/xrpc/com.atproto.identity.resolveHandle?handle=<handle>"` |

## Port 22 is not optional

Pushing to a knot only works over SSH. The HTTP endpoint answers, verbatim:

```
Pushes are only supported over SSH.
```

And the appview builds clone URLs as `git@<hostname>:<owner>/<repo>`, with no way to advertise a
different port. So `<hostname>:22` has to reach the pod, or the knot is read-only.

The chart ships a second Service, `knot-ssh`, of type `LoadBalancer` for this. Swap it for a
NodePort, or drop it and use a `hostPort`, if that does not fit your cluster. Both that Service and
your HTTPS ingress have to answer on the same DNS name.

> [!IMPORTANT]
> The appview verifies the knot over the public internet with an SSRF-protected dialer that refuses
> private addresses. A knot only reachable on an RFC1918 address cannot be verified.

## What must persist

Three volumes, all of them irreplaceable except the last one's convenience:

| Volume | Path | Why |
|---|---|---|
| `repositories` | `/home/git/repositories` | the bare git repos, one directory per repo DID |
| `server` | `/app` | SQLite: known DIDs, public keys, repo mapping, ACL, jetstream cursor |
| `keys` | `/etc/ssh/keys` | SSH host keys, generated on first boot |

Lose `keys` and every client gets `REMOTE HOST IDENTIFICATION HAS CHANGED`, because the image
regenerates the three host keys when the directory is empty.

## The image

`atcr.io/tangled.org/knot` is the community image, but that registry requires authentication even to
pull. This chart therefore points at `ghcr.io/qjoly/knot`, rebuilt from the exact same
[knot-docker](https://tangled.org/tangled.org/knot-docker) Dockerfile by
[`.github/workflows/knot-image.yaml`](.github/workflows/knot-image.yaml), for `linux/amd64` and
`linux/arm64`. To build another upstream tag:

```bash
gh workflow run knot-image.yaml -f ref=v1.16.1-alpha
```

## Things worth knowing before you tune it

- **One replica, always.** A single SQLite file backs both the application and the casbin enforcer,
  and the boot sequence rewrites the hooks of every repository.
- **Runs as root, on purpose.** s6 supervises the knot server *and* sshd, and sshd binds port 22.
  The knot process itself is dropped to the `git` user (uid/gid 1000) by `s6-setuidgid` — hence
  `fsGroup: 1000` on the volumes.
- **Do not move `KNOT_SERVER_INTERNAL_LISTEN_ADDR`.** The internal API on `127.0.0.1:5444` has no
  authentication at all: it can authorise pushes, dump every user's public key, and serves pprof.
  It is loopback by design. On top of that, the image's sshd config hardcodes `localhost:5444` in
  its `AuthorizedKeysCommand`, so changing the port breaks SSH auth while HTTP keeps working.
- **`KNOT_SERVER_ADMIN_SECRET` is left empty.** `/admin` is served on the *public* port behind HTTP
  basic auth only; while the secret is unset the middleware rejects everything. Set it only if you
  also block `/admin` at the ingress.
- **`/events` is a WebSocket** on 5555. Your ingress needs upgrade support and a generous read
  timeout on that path.
- **Secure mode is off.** `KNOT_SERVER_SECURE_MODE=true` needs `CAP_SETUID`/`CAP_SETGID`/`CAP_CHOWN`,
  a kernel with Landlock V2, and a one-off `knot migrate-isolation` run, or the server refuses to
  boot.
