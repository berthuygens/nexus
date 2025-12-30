# Cloudflare Worker Setup Guide

This guide will help you set up the OAuth worker for automatic Google Calendar token refresh.

## Prerequisites

- Cloudflare account (free tier is fine)
- Node.js installed (for wrangler CLI)
- Google Cloud Console access

## Step 1: Install Wrangler CLI

```bash
npm install -g wrangler
```

## Step 2: Login to Cloudflare

```bash
wrangler login
```

This opens a browser to authenticate with your Cloudflare account.

## Step 3: Create KV Namespace

KV (Key-Value) storage is used to store the refresh token securely.

```bash
cd cloudflare-worker
wrangler kv:namespace create "OAUTH_TOKENS"
```

This will output something like:
```
{ binding = "OAUTH_TOKENS", id = "xxxxxxxxxxxxxxxxxxxx" }
```

Copy the `id` value and update `wrangler.toml`:

```toml
[[kv_namespaces]]
binding = "OAUTH_TOKENS"
id = "YOUR_KV_NAMESPACE_ID_HERE"
```

## Step 4: Update Google OAuth Settings

Go to [Google Cloud Console](https://console.cloud.google.com/apis/credentials) and update your OAuth 2.0 Client:

1. Click on your existing OAuth Client ID
2. Add to **Authorized redirect URIs**:
   ```
   https://sysopbeta-oauth.YOUR_SUBDOMAIN.workers.dev/callback
   ```
   (You'll get the exact URL after first deploy)

3. **Important**: Note down your **Client Secret** (click "DOWNLOAD JSON" or copy from the page)

## Step 5: Set Secrets

```bash
wrangler secret put GOOGLE_CLIENT_ID
# Paste your Google Client ID when prompted

wrangler secret put GOOGLE_CLIENT_SECRET
# Paste your Google Client Secret when prompted
```

## Step 6: Deploy Worker

```bash
wrangler deploy
```

This will output your worker URL, something like:
```
https://sysopbeta-oauth.your-subdomain.workers.dev
```

**Important**: Go back to Google Cloud Console and add this exact URL + `/callback` to your Authorized redirect URIs!

## Step 7: Test the Worker

1. Open your worker URL `/auth` endpoint:
   ```
   https://sysopbeta-oauth.your-subdomain.workers.dev/auth
   ```

2. You should be redirected to Google login
3. After authorizing, you'll see "Connected!" and the window will close

4. Test the token endpoint:
   ```bash
   curl https://sysopbeta-oauth.your-subdomain.workers.dev/token
   ```
   Should return an access token!

## Step 8: Update Dashboard

Update your dashboard's `index.html` to use the worker. See the main README for frontend changes.

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth` | GET | Redirects to Google OAuth consent screen |
| `/callback` | GET | Handles OAuth callback (don't call directly) |
| `/token` | GET | Returns fresh access token |
| `/status` | GET | Check if authenticated |
| `/logout` | POST | Remove stored tokens |

## Troubleshooting

### "No refresh token stored"
- Go to `/auth` to authenticate first
- Make sure you completed the Google OAuth flow

### "invalid_grant" error
- The refresh token has been revoked
- Go to `/auth` to re-authenticate

### CORS errors
- Check that your GitHub Pages URL is in the `allowedOrigins` array in `worker.js`

### "redirect_uri_mismatch" from Google
- Make sure the callback URL in Google Console exactly matches your worker URL + `/callback`

## Security Notes

- Refresh tokens are stored in Cloudflare KV (encrypted at rest)
- Client secret is stored as a Cloudflare secret (never exposed)
- CORS is restricted to your domain only
- This is a single-user setup; for multi-user, you'd need to add user identification
