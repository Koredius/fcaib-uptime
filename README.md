# fcaib-uptime

External availability check for https://fcaib.edu.ng, run by GitHub Actions every 10 minutes from outside the server.

It exists because on 2026-11-13 the site was down for about 80 hours and nothing noticed. The server now runs its own watchdog, which restarts NGINX and alerts on Telegram, but a watchdog on the server cannot report the server itself being down. This repository covers that case.

## How it works

- `/.github/workflows/uptime.yml` fetches `/api/health` up to 3 times, 20 seconds apart, and passes only if the response says `"status":"healthy"` and `"database":"connected"`.
- A run fails only when the site is down. The workflow's own run history is the state: the first failed run sends **DOWN**, every 12th consecutive failure (about 2 hours) sends **STILL DOWN**, and the first passing run after a failure sends **RECOVERED** with the downtime.
- A keepalive job makes an empty commit when the repository has been quiet for 45 days, because GitHub switches off scheduled workflows after 60 days without activity.

GitHub runs scheduled workflows on a best-effort basis, so checks can start several minutes late. Detection within roughly 10 to 20 minutes is the realistic expectation.

## Secrets

`TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`, the same chat the server watchdog uses.

## Testing the alert path

Actions, FCAIB uptime, Run workflow, and give `url` a path that fails, for example `https://fcaib.edu.ng/api/no-such-route`. You get a `[TEST]` DOWN message. Run it again with `url` empty and you get a RECOVERED message.
