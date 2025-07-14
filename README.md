# RSS intelligence digest

This workflow polls one or more public RSS/Atom feeds, removes old or irrelevant entries, deduplicates them, and produces a compact digest. Slack delivery is optional; the core workflow needs no account or API key.

## Architecture

`Webhook or disabled schedule → normalize → test fixture or RSS Read → deterministic filter → digest → mock/Slack/skip → receipt`

The filtering step is deliberately deterministic rather than LLM-based. It makes scheduled runs predictable, keeps the production path free, and avoids sending third-party article text to an AI provider. An optional Gemini enrichment stage can be inserted after `Select RSS Items` if semantic clustering is later required.

## Run the included test

After the root `npm run local:setup` and `npm run local:start` commands:

```powershell
$body = Get-Content -Raw .\5-rss-intelligence-digest\fixtures\rss-digest.json
Invoke-RestMethod -Method Post -Uri http://localhost:5678/webhook/rss-intelligence -ContentType application/json -Body $body
```

Expected: `status=completed`, `item_count=2`, `test_mode=true`, and `notification.status=mocked`. The fixture proves keyword inclusion, exclusion, time-window filtering, and duplicate suppression without making an external request.

## Production configuration

- Set `RSS_FEED_URLS` to comma-separated HTTPS feed URLs or supply `feed_urls` in the webhook payload.
- Set `filters.keywords`, `exclude_keywords`, `lookback_hours`, and `max_items` per run when needed.
- Add `SLACK_WEBHOOK_URL` only if Slack delivery is wanted.
- Enable `Daily RSS Schedule` only after a real feed has been tested manually.

For exactly-once notifications, persist item IDs in a small PostgreSQL table and exclude previously delivered IDs. Add feed allowlisting and outbound network controls when workflow callers can supply URLs; otherwise the RSS node could be used for server-side request forgery.

## Free-tier notes

Public RSS/Atom feeds normally need no key, but each publisher's terms still apply. Slack's free plan can receive app/webhook messages; workspace retention and app limits are separate from this workflow.
