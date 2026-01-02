# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DAEMON Dashboard is a personal developer dashboard - a static single-page application hosted on GitHub Pages with a Cloudflare Worker backend for OAuth token management. It displays GitHub issues/PRs, Google Calendar events, quick links, and Reddit posts.

## Development Commands

```bash
# Start local development server
python3 -m http.server 8080
# Then open http://localhost:8080

# Deploy Cloudflare Worker (from cloudflare-worker directory)
cd cloudflare-worker
wrangler deploy

# Set Worker secrets
wrangler secret put GOOGLE_CLIENT_ID
wrangler secret put GOOGLE_CLIENT_SECRET

# Create KV namespace (if needed)
wrangler kv:namespace create "OAUTH_TOKENS"
```

## Architecture

```
Frontend (Static)                    Backend (Cloudflare Worker)
├── index.html (entire app)          ├── worker.js (OAuth handler)
│   ├── CSS styles                   │   ├── /auth → Google OAuth redirect
│   ├── HTML structure               │   ├── /callback → Token exchange
│   └── JavaScript logic             │   ├── /token → Fresh access token
│       ├── GitHub API calls         │   ├── /status → Auth check
│       ├── Calendar API calls       │   └── /logout → Remove tokens
│       ├── Reddit API calls         └── wrangler.toml (config)
│       └── LocalStorage (settings)
```

**Key architectural decisions:**
- Single HTML file contains all CSS, HTML, and JavaScript (no build system)
- GitHub token stored in browser localStorage
- Google refresh token stored in Cloudflare KV (server-side)
- Worker handles OAuth token refresh automatically - users authenticate once

## External APIs

- **GitHub REST API**: Issues and PRs (requires `repo`, `read:user` scopes)
- **Google Calendar API v3**: Calendar events (requires `calendar.readonly` scope)
- **Reddit JSON API**: Posts from SFW subreddits (no auth needed)

## Calendar Event Filtering

Events with these titles are automatically filtered (work locations):
- home, thuis, office, kantoor, bureau, HT BXL, VAC GENT

## Worker Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth` | GET | Redirect to Google OAuth |
| `/callback` | GET | OAuth callback handler |
| `/token` | GET | Get fresh access token |
| `/status` | GET | Check authentication status |
| `/logout` | POST | Remove stored tokens |

## Live URL

https://berthuygens.github.io/daemon/
