# Deploy LunaTV from this GitHub repository

This folder prepares a personal, self-hosted LunaTV instance. **GitHub hosts the code
and builds the container image; GitHub Pages cannot run this Next.js server and Redis.**
You still need a Docker host or a Docker-compatible hosting provider.

## 1. Publish your own image with GitHub Actions

1. Merge the deployment pull request into `main`.
2. In `https://github.com/kitzhn/LunaTV/actions`, enable GitHub Actions if GitHub
   prompts you to do so on this fork.
3. Open **Build and publish LunaTV image**, select **Run workflow** on `main`
   (or push a commit to `main`).
4. Confirm the workflow succeeds and the image `ghcr.io/kitzhn/lunatv:latest`
   exists under your GitHub Packages.

The workflow builds Linux AMD64 and ARM64 images. Pull-request checks **build without
publishing**. Pushing to `main` publishes to your own GHCR namespace instead of
trying to overwrite `ghcr.io/moontechlab/lunatv`.

**GHCR package access:** A newly created package might be private. If it is, either
adjust its visibility under GitHub Packages or authenticate Docker to GHCR using
a correctly scoped token. Never commit tokens or credentials to Git.

## 2. Start on a Docker server or your own PC

Install Docker Engine + Docker Compose v2 (Docker Desktop on Windows also works).
Then on the Docker machine:

```sh
git clone https://github.com/kitzhn/LunaTV.git
cd LunaTV/deploy
cp .env.example .env
# Edit .env: set a long, unique LUNATV_PASSWORD (required).
docker compose config
docker compose pull
docker compose up -d
docker compose ps
```

If running on Windows PowerShell, replace `cp` with `Copy-Item` if preferred.
When running Docker on your local PC, visit `http://127.0.0.1:3000`.
The compose file intentionally binds the app to localhost only: it is **not**
reachable from other devices until you arrange appropriate network access.

For a Linux server, an SSH tunnel is one safe testing method:

```sh
ssh -L 3000:127.0.0.1:3000 user@your-server
```

Then open `http://127.0.0.1:3000` on your computer. For permanent remote use,
configure a trusted HTTPS reverse proxy and authentication, without exposing
Redis to the public Internet. If container image pull is denied, check GHCR
visibility/authentication before retrying.

## 3. Configure and maintain

- Log in with the administrator username and password from `.env`.
- Add only content providers you are authorized to access; LunaTV contains no
  built-in media catalog or playback sources.
- Keep registration disabled for a private deployment and do not share the
  administration URL publicly.
- To update: `docker compose pull && docker compose up -d`.
- Back up the Docker volume `redis-data` and your private `.env` **securely**.
  The Redis volume uses append-only persistence; this improves durability but
  is not a substitute for backups.
- For reproducible upgrades, replace the `:latest` image tag in
  `docker-compose.yml` with a tested SHA/version.

## Important caveats

- The current fork's application code may lag the upstream
  `MoonTechLab/LunaTV`. The GitHub Actions image is built from **this fork**;
  updating to upstream is a separate review and may involve license changes.
- The upstream README currently recommends Docker-based deployment. Free GitHub
  Actions build minutes do **not** imply free 24/7 web hosting.
- This PR only prepares reproducible deployment: no Docker service has been
  started, and no public site or domain is automatically provisioned.
