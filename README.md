# Free — a schema playground for agents

**Free** is a sandbox app for TheItemApp. It exists so AI agents (and people) can
spin up **disposable data models on the fly** and visualize them with the
platform's built-in **chart / pivot / table / calendar / kanban / gallery**
prefabs — *without contaminating any other app's schema*.

The motivating use case:

> "Agent, what's the weather this week?" → the agent fetches the data from a
> weather MCP, creates a `free_weather_week` model here, drops the 7 rows in,
> spins up a chart + a table window, and renders them inline in chat.

Everything an agent creates here is namespaced to the Free app (`appId
910000000000000000000001`) and lives in `free_*` collections, so it never
touches production models.

## What ships in the seed

A **seeds-only satellite** — no backend, no federated frontend. All UI is core's
built-in prefabs. The seed contains:

| Collection | What |
|---|---|
| `apps` | The Free app record (icon, AI system prompt describing the playground loop). |
| `items` | `free_demo_weather` — an example 7-day weather model. The canonical template. |
| `free_demo_weather` | 7 rows of demo data. |
| `item_chart_views` | "Weekly Temperature" line chart (high/low °C over the week). |
| `dashboards` / `windows` | The **Free Playground** home dashboard rendering the demo model three ways: chart, pivot, table. |
| `coding_agent_skills` / `chat_skills` | The `free-playground-visualization` skill — copy-paste templates teaching both the coding agent and chat how to use Free. |

## The agent loop

1. **Create a model** — `data.create('items', …)` with `key`/`collection`
   prefixed `free_`, `appId` Free, a JSON-schema, `api.crud`, and an `auth` block.
2. **Add data** — the collection is live the instant the items record is written.
   Insert rows via the dynamic HTTP API (`POST /api/dynamic/free_<name>`) — a
   brand-new collection isn't in the theitemapp MCP `modelKey` enum until the
   next session, but the HTTP API works immediately.
3. **Visualize** — create `windows` bound to the model with a built-in prefab
   (pivot is fully inline; chart needs an `item_chart_views` companion).
4. **Show it** — reference a `windows/<id>` inline in chat (it renders live), or
   point the user at a dashboard.
5. **Clean up** — delete throwaway `free_*` models when done.

See the `free-playground-visualization` skill for the full, copy-paste workflow.

## Deploy

Seeds-only, joins core's `theitemapp` docker network:

```bash
docker compose up -d --build      # builds free-seeds, registers with core, seeds
```

Requires `APP_REGISTRATION_KEY` in `.env` (see `.env.example`). The base image
`seed-server:1.0.0` is inherited from core.

## Namespace

- **App id:** `910000000000000000000001`
- **ObjectId prefix:** `910000000000000000…`
- **Collections:** `free_*` (agent-created models), plus the demo `free_demo_weather`.
