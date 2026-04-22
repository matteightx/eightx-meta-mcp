# eightx-meta-mcp

> A **read-only** Model Context Protocol (MCP) server for Meta Ads (Facebook + Instagram) and Instagram Business — built so an AI agent can pull campaign performance, creative-level metrics, audience insights, and competitor ads from the Meta Ad Library, without ever being able to spend a dollar or change a setting.

Built and maintained by **[8x](https://eightx.co)**. We help operators automate marketing and finance ops with AI agents. If you want help deploying this, customizing it, or wiring it into a wider AI workflow, [book a free diagnostic call](https://calendly.com/d/csm9-53t-pg6/diagnostic-strategy-call).

---

## What it does

- **12 read-only tools** an LLM can call: ad account overview, campaign performance, ad set listings, individual ad performance, Ad Library competitor research, plus 6 Instagram Business tools (account info, media list, per-post insights, account-level insights, audience demographics, targeting search).
- **No write tools.** No `create_*`, no `update_*`, no `pause_*`, no `upload_*`, no budget changes. Safe to give to any AI agent — the worst it can do is tell you the truth.
- Single Meta access token covers everything you have permission for. No per-account configuration.
- Streamable HTTP MCP transport, ready to deploy on Railway in one click.

---

## Tools

### Meta Ads (6)

| Tool | What it returns |
|---|---|
| `list_ad_accounts` | All ad accounts your token has access to (id, name, status, currency, spend, balance) |
| `get_account_overview` | High-level spend / impressions / clicks / CTR / CPC / CPM / conversions / ROAS for an account over a date range |
| `get_campaign_performance` | Same metrics, broken down by campaign with status and objective |
| `list_ad_sets` | All ad sets in an account or campaign with budgets, optimization goal, billing event, targeting summary |
| `get_ad_performance` | Per-ad breakdown — spend, CTR, CPC, conversions, sortable, useful for finding top/bottom creatives |
| `search_ad_library` | Active ads by any advertiser on Facebook & Instagram. Pass a `search_term`, a `page_id`, or both. Pure competitor research. |

### Instagram Business (5)

| Tool | What it returns |
|---|---|
| `get_ig_account` | Look up the IG Business Account ID linked to a Facebook Page (followers, bio, etc.) |
| `get_ig_media` | Recent posts and reels with basic engagement counts |
| `get_ig_media_insights` | Per-post detail incl. video retention for reels (plays, reach, saves, shares, watch time) |
| `get_ig_account_insights` | Account-level metrics over a date range (reach, profile views, total interactions, etc.) |
| `get_ig_audience` | Audience demographics: age, gender, top cities, top countries (requires 100+ followers) |

### Targeting (1)

| Tool | What it returns |
|---|---|
| `search_targeting` | Search Meta's targeting database for interests, geos, employers, etc. — read-only research only |

---

## Why this exists

When you put an AI agent in front of Meta Ads, the worst-case scenario is "agent decides to launch a million-dollar campaign in the wrong country." So the safest first integration is one where it **can't do anything bad**. This MCP gives an LLM the same view a junior media buyer would have — full visibility into performance — with none of the buttons that change anything.

That's also exactly the right shape for an "AI CFO" that needs to talk about marketing ROI without becoming a media buyer.

If you need write access (create campaigns, ad sets, ads, lead forms, upload creatives) with proper safety rails — naming conventions, allowed countries, always-PAUSED creation, etc. — 8x runs a production version. [Get in touch.](https://calendly.com/d/csm9-53t-pg6/diagnostic-strategy-call)

---

## Setup — get a Meta access token

You need one **long-lived access token** that has access to the ad accounts and Pages you care about.

1. Go to <https://developers.facebook.com/apps> → create an app (or use an existing one).
2. In **App Roles → Roles**, make sure your Facebook user is listed as a developer or admin.
3. Open **Tools → Graph API Explorer**.
4. Pick your app, then click **Generate Access Token**. Add these scopes:
   - `ads_read`
   - `business_management`
   - `pages_read_engagement`
   - `instagram_basic`
   - `instagram_manage_insights`
   - `read_insights`
5. Click **Generate**. Copy the short-lived token.
6. Convert it to a long-lived (60-day) token using the Access Token Debugger or the long-lived endpoint. (See Meta's docs.)
7. Set it as `META_ACCESS_TOKEN` in your environment.

---

## Deploy to Railway

1. Deploy this repo to Railway. (New Project → Deploy from GitHub repo.)
2. Set env var `META_ACCESS_TOKEN` to your long-lived token.
3. Railway sets `PORT` automatically.
4. Open `https://<your-service>.up.railway.app/health` to confirm the token is valid.
5. Wire up Claude / your MCP client:
   ```json
   {
     "mcpServers": {
       "meta-ads": {
         "url": "https://<your-service>.up.railway.app/mcp"
       }
     }
   }
   ```

---

## Run locally

```bash
git clone https://github.com/matteightx/eightx-meta-mcp.git
cd eightx-meta-mcp
npm install
cp .env.example .env
# Fill in META_ACCESS_TOKEN
npm start
# server runs on http://localhost:3000
```

For Claude Code over stdio, you can wire the HTTP server up via an MCP HTTP shim, or run the server directly and point Claude Code at `http://localhost:3000/mcp`.

---

## Configuration reference

| Env var | Default | Notes |
|---|---|---|
| `META_ACCESS_TOKEN` | — | Required. Long-lived token with the scopes listed above. |
| `META_API_VERSION` | `v21.0` | Meta Graph API version. Bump as needed. |
| `PORT` | `3000` | HTTP port. Railway sets automatically. |

---

## Need write access, custom dashboards, or a full AI marketing ops setup?

This is the open-source, read-only version. The team at **8x** built it and runs the production version with write support, naming-convention enforcement, country/age safety rails, batch-creative workflows, and AI agents that actually run media for you.

- **[eightx.co](https://eightx.co)** — what we do
- **[Book a free diagnostic call](https://calendly.com/d/csm9-53t-pg6/diagnostic-strategy-call)** — tell us about your ads stack and we'll show you what's possible

PRs welcome. Issues welcome. License: MIT.
