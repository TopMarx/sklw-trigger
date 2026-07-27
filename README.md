# sklw-trigger

Trigger-only repo for [SKLW](https://sklw.vercel.app). Every 5 minutes a
GitHub Actions workflow calls `GET /api/cron/refresh?mode=auto` with a bearer
token. The endpoint is fully self-gating - it decides whether any work is due
and returns in under a second when idle - so this repo is nothing but a dumb
clock. It contains no league data and no secrets.

## Why a separate public repo?

The SKLW app lives in a private repo, and GitHub bills Actions minutes on
private repos (a 5-minute cadence is roughly 8,600 minutes/month against a
2,000-minute free allowance). Public repos get unlimited free minutes on
standard runners, so the trigger lives here on its own.

## Setup

Two commands, run once by the operator:

```sh
op read op://sklw/Development/CRON_SECRET | gh secret set CRON_SECRET --repo TopMarx/sklw-trigger
gh variable set SKLW_URL --repo TopMarx/sklw-trigger --body "https://sklw.vercel.app"
```

`CRON_SECRET` must match the value configured in the Vercel production
environment. Until both are set, the workflow no-ops green rather than
failing every 5 minutes.

## Timing caveat

GitHub Actions cron is best-effort: at peak load ticks are routinely delayed
(up to ~15 minutes) and occasionally skipped. That is fine here - the route,
not the scheduler, decides what work is due, so a late tick just does that
work a few minutes later.

## Keepalive

GitHub disables scheduled workflows after 60 days without repo activity, and
this repo never changes. `keepalive.yml` makes an empty commit once a month
to keep the schedule enabled. It pushes with the built-in `GITHUB_TOKEN`,
which does not trigger other workflows, so there is no recursion.

## Pausing the trigger

Actions tab -> "Refresh tick" -> "..." menu -> Disable workflow. Re-enable
the same way. (Deleting the repo also works, but then you lose the history
of runs.)

## Manual runs

Actions tab -> "Refresh tick" -> "Run workflow" lets you pick a mode
(`auto`, `capture`, `live`, `confirmed`, `bootstrap`) and optionally add
`force=1` for operator-forced passes.
