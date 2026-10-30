# P0rtalz YouTube verification

The clean flow is **Discord /verify → Worker → Google consent → YouTube check → Kite role assignment**. There is no intermediate landing page; the Worker redirects straight to Google.

## One-time setup

Add these repository Actions secrets under **Settings → Secrets and variables → Actions → New repository secret**:

- `CLOUDFLARE_API_TOKEN` — token with permission to edit/deploy Workers.
- `CLOUDFLARE_ACCOUNT_ID` — your Cloudflare account ID.
- `GOOGLE_CLIENT_ID` — the OAuth client ID.
- `GOOGLE_CLIENT_SECRET` — the OAuth client secret.
- `STATE_HMAC_SECRET` — a long random value (generate one with `openssl rand -hex 32`).
- `KITE_WEBHOOK_URL` — the private URL of the Kite `verification` webhook. Keep it secret.

Then run **Actions → Deploy verification worker → Run workflow** (or push a change to `worker/`).

## Finish OAuth setup

1. Copy the deployed Worker URL from the Cloudflare dashboard. It will look like `https://p0rtalz-verification.<your-workers-subdomain>.workers.dev`.
2. In Google Cloud Console, add this exact authorized redirect URI to the OAuth client: `<WORKER_URL>/auth/callback`.
3. In Discord/Kite, set the `/verify` link button URL to `<WORKER_URL>/auth/start?discord_user_id={{user.id}}`.

The Worker URL is account-specific, so it must be filled in after the first deployment. Do not put secrets in this repository or in the browser page.

## Endpoints

- `/health` — deployment check.
- `/auth/start?discord_user_id=...` — validates the Discord ID and redirects directly to Google.
- `/auth/callback` — exchanges the OAuth code, checks the P0rtalz subscription, and calls the Kite webhook.

Google OAuth needs the YouTube read-only scope. Users whose YouTube subscriptions are private may not be verifiable through the subscriptions API.
