# Allio

> **Note:** The source code of Allio lives in a private repository. This public repo is a
> project overview: what Allio is, why I built it, and what I built in it.

Allio is a mobile app for creating and managing your own AI bot agents from your phone.
You define a bot for one specific job, give it the capabilities it needs, and let it work on
its own. When it's time, it runs, does the job, and drops the result onto your dashboard as a
report. You get a push notification, and if you want, it's posted to Slack too.

## Why

Most AI assistants wait for you to ask them something. I wanted the opposite: a small team of
focused agents that each own a recurring job (a daily summary, watching a GitHub repo,
preparing a weekly report) and just get it done. And I wanted to manage all of them from my
phone.

Allio is built around three ideas:

- **One bot, one job.** Every bot has its own persona, capabilities, schedule and memory.
  Bots are isolated from each other and never see each other's data.
- **The phone is the control panel.** You create, run, pause and chat with bots from a mobile
  app. Results come to you; you don't have to go looking for them.
- **Work should land where you already are.** Reports go to an in-app board, to push
  notifications and to Slack.

## What I built

### Mobile app (React Native / Expo)

- **Board:** the home screen is a board of draggable sticky notes. Reports from bots show up
  here as notes. A note that starts with `@bot-name` becomes a task for that bot.
- **Bots:** create a bot from a ready-made template or from scratch, pick its capabilities,
  and see its status, reports and history.
- **Tasks:** a list of what each bot will do and when it will run next, with pause and resume.
- **Chat:** talk to any bot directly. Answers are rendered as Markdown with cited sources.
- **Slack view:** read channels and reply to messages without leaving the app.
- **Apps:** one-tap integrations (GitHub, Google Calendar, Slack) that set up a dedicated bot.
- **Settings:** usage and token consumption overview, connection status, version.
- A custom dark theme, a slide-out navigation drawer, attachments and push notifications.

### Backend (Node.js / Express / MongoDB)

- **Bot runner:** a single path that runs a bot, shared by "run now" and scheduled runs.
- **Scheduler:** my own cron engine with time-zone support, which also catches up on missed
  runs after a restart.
- **Secure by design:** API keys never reach the phone. The server issues a short-lived,
  narrowly scoped token for each bot on every request.
- **Signed webhooks:** completion events are checked with an HMAC signature and deduplicated,
  then trigger a push notification and a Slack message.
- **MCP integrations:** GitHub, Google Calendar and Slack are connected through the Model
  Context Protocol. Integrations are read-only by default.
- **Custom MCP bridges:** I wrote my own MCP servers:
  - a **Slack bridge** that turns an incoming webhook into a tool the agent can call,
  - a **reader bridge** that fetches a web page and extracts its clean, readable text, so an
    agent can read a full article instead of a screenshot of it.
- **Slack Events API:** when someone writes in a channel, the server verifies the signature,
  sends a push notification and pins a note to the board.
- **Usage tracking:** estimated token usage per turn and per conversation.

## Architecture

```
  ┌───────────────┐   REST    ┌──────────────────┐  short-lived  ┌──────────────┐
  │  Mobile app   │──────────▶│  Allio server    │──────────────▶│  AI agent    │
  │  (Expo)       │           │  Express + Mongo │    token      │  platform    │
  └───────▲───────┘           └───────▲──────────┘               └──────┬───────┘
          │                           │       signed webhook            │
          │      push notification    └─────────────────────────────────┘
          └───────────────────────────────────┐
                                              ▼
                                       ┌─────────────┐
                                       │    Slack    │
                                       └─────────────┘
```

A bot is a database record, not a separate service. Each bot gets its own isolated space
(documents, memory, scheduled tasks), so many bots can share one setup without ever seeing each
other's data.

## Tech stack

| Layer | Technologies |
|---|---|
| Mobile | React Native, Expo, expo-router, expo-notifications |
| Backend | Node.js, Express, Mongoose |
| Database | MongoDB |
| Integrations | Model Context Protocol (MCP), Slack (Webhooks + Events API), GitHub, Google Calendar, Expo Push |

## Status

Allio is a personal project under active development. The source code is private. Feel free
to reach out if you'd like to know more.
