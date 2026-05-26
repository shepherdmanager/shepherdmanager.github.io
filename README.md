# shepherdmanager.github.io

GitHub Pages vanity redirect for **Shepherd Manager** — a pastoral care PWA.

## How it works

`index.html` contains a `<meta http-equiv="refresh">` tag that immediately redirects visitors to the app hosted on a Tailscale Funnel endpoint. GitHub Pages serves the redirect; the NAS serves the app.

```
User visits shepherdmanager.github.io
  → GitHub Pages serves index.html
    → Browser redirects to https://atlas-server.taila932a4.ts.net/
      → Tailscale Funnel (TLS) → Caddy → shepherd-app container
```

## Setup (replicable for any app)

This pattern gives any app a clean `<org>.github.io` URL without a custom domain or DNS changes.

### 1. Create a GitHub org

Go to https://github.com/organizations/plan and create a free org matching the desired URL (e.g., `myappname` → `myappname.github.io`).

### 2. Create the redirect repo

```bash
mkdir /tmp/myapp-redirect && cd /tmp/myapp-redirect

cat > index.html << 'HTML'
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="refresh" content="0; url=https://atlas-server.taila932a4.ts.net/<path>">
  <title>My App</title>
</head>
<body><p>Redirecting to <a href="https://atlas-server.taila932a4.ts.net/<path>">My App</a>...</p></body>
</html>
HTML

git init && git add . && git commit -m "Redirect page"
gh repo create <org>/<org>.github.io --public --source=. --push
```

Replace `<org>` with your GitHub org name and `<path>` with the Caddy route (empty for root, `magnifica/` for a subpath app).

### 3. Enable GitHub Pages

Usually auto-enabled for `*.github.io` repos. If not:

```bash
gh api repos/<org>/<org>.github.io/pages -X POST \
  --input - <<< '{"build_type":"legacy","source":{"branch":"main","path":"/"}}'
```

### 4. Verify

Wait ~2 minutes for GitHub Pages deploy, then:

```bash
curl -sL -o /dev/null -w '%{url_effective}\n' https://<org>.github.io/
```

## Infrastructure context

This repo is part of the NAS Docker migration infrastructure:

| Component | Location | Purpose |
|---|---|---|
| Tailscale Funnel | atlas-server NAS | TLS termination, public ingress |
| Caddy | `/volume1/docker/caddy/` | Path-based reverse proxy |
| App containers | `/volume1/docker/<app>/` | Individual Docker Compose stacks |
| GitHub Pages orgs | `<org>.github.io` repos | Vanity URL redirects |
| Dockge | `http://atlas-server:5001` | Container management UI (LAN only) |
