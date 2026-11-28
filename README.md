# deploy-rakazo

A skill for deploying [Rakazo](https://github.com/elie222/rakazo) — an open-source platform for persistent AI teammates with memory, routines, and their own computers — on [Railway](https://railway.com) using the official one-click template.

## What it covers

- Railway template deployment (web, api, worker, postgres)
- Environment variable configuration (OpenRouter, sandbox providers, auth secrets)
- Post-deploy verification (first user registration, health check)
- Common issues and troubleshooting

## Template

- **Template URL:** https://railway.com/deploy/rakazo
- **Deploy URL:** https://railway.com/new/template/rakazo
- **Source:** https://github.com/elie222/rakazo
- **Image:** ghcr.io/elie222/rakazo/app:v0.3.3

## Usage

This skill is designed for AI agents. When a user asks to deploy Rakazo on Railway, the agent follows the steps in `SKILL.md`:

1. Open the Railway template
2. Sign in to Railway (GitHub/Google/email)
3. Review and configure environment variables
4. Deploy all four services
5. Verify post-deployment (register first user, check health)

## Retrieve on another host

The skill also covers taking over an existing deployment from another WSL/Linux host:
sync `https://github.com/phycomp/deploy-rakazo.git` (`main`), `railway link` with
the existing project ID (never `railway init`), read web URL via `railway domain`,
check `https://<domain>/health`, and read secrets from Railway Variables into
`~/.config/rakazo/.env` (600). Save a secret-free handoff note for the next host.
See the "Retrieve an Existing Deployment (other host)" section in `SKILL.md`.

## Logic check

- RED baseline recorded three natural failures: guessing project ID/domain/health path,
  wanting to scrape shell history for tokens, and assuming the skill lives in
  `phycomp/oskill`.
- GREEN adds a Retrieve section addressing exactly those failures: own-repo pull,
  `railway link` over `init`, documented `/health`, Railway Variables as the only
  secret source, and an immutable `ENCRYPTION_KEY` rule.
- REFACTOR: red-flag table closes the four recurring rationalizations; add a
  missing-CLI guard so a fresh host does not stall at `railway: command not found`.

## License

This skill documents the deployment process for Rakazo. Rakazo itself is licensed under its own terms — see https://github.com/elie222/rakazo for details.
