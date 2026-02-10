# StravaChat

An MCP server that connects Claude to the Strava API — ask about your training data in natural language.

---

## What It Does

StravaChat gives Claude direct access to your Strava account through 8 tools:

| Tool | Description |
|------|-------------|
| `get_athlete_profile` | Your profile — name, location, bio |
| `get_athlete_stats` | Lifetime, YTD, and recent totals for run/ride/swim |
| `list_activities` | Browse activities with pagination and date filters |
| `get_activity` | Deep dive into a single activity (splits, segments, metrics) |
| `get_activity_laps` | Lap-by-lap breakdown (pace, HR, distance) |
| `get_activity_zones` | Time in each heart rate / power zone |
| `list_routes` | Your saved routes |
| `get_route` | Route details (distance, elevation, waypoints) |

### Example Prompts

> "How far did I run this month?"

> "Show me my last 5 rides and compare average speed."

> "What was my heart rate zone distribution on yesterday's run?"

> "From my longterm Strava stats, what do you recommend me to improve my running performance?"

---

## Setup

### Prerequisites

- **Node.js** >= 18
- A **Strava account** with API access
- **Claude Desktop** or **Claude Code**

### 1. Clone and install

```bash
git clone <repo-url> && cd stravachat
npm install
```

### 2. Create a Strava API application

1. Go to [strava.com/settings/api](https://www.strava.com/settings/api)
2. Create an application (or use your existing one)
3. Set **Authorization Callback Domain** to `localhost`
4. Note your **Client ID** and **Client Secret**

### 3. Configure credentials

```bash
cp .env.example .env
```

Edit `.env` and fill in your Client ID and Client Secret:

```
STRAVA_CLIENT_ID=12345
STRAVA_CLIENT_SECRET=abc123...
STRAVA_REFRESH_TOKEN=
```

### 4. Get a refresh token

The token from the Strava settings page is often expired. Use the included helper to get a fresh one:

```bash
npx tsx scripts/get-token.ts
```

This will:
1. Print an authorization URL — open it in your browser
2. You authorize the app on Strava
3. The script prints a valid refresh token

Paste the token into your `.env`:

```
STRAVA_REFRESH_TOKEN=<token from script>
```

### 5. Build

```bash
npm run build
```

### 6. Connect to Claude

Add to your Claude configuration:

**Claude Desktop** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "strava": {
      "command": "node",
      "args": ["/absolute/path/to/stravachat/dist/index.js"]
    }
  }
}
```

**Claude Code** (`~/.claude.json`):

```json
{
  "mcpServers": {
    "strava": {
      "command": "node",
      "args": ["/absolute/path/to/stravachat/dist/index.js"]
    }
  }
}
```

Restart Claude. You're ready to go.

---

## Development

```bash
npm run dev      # Watch mode — recompiles on save
npm run build    # One-time build
npm start        # Run the server directly
```

### Project Structure

```
src/
  index.ts            Entry point — server + stdio transport
  strava-client.ts    Strava API client with auto token refresh
  tools.ts            MCP tool definitions (8 tools)
scripts/
  get-token.ts        One-time OAuth helper
```

---

## Notes

- **Token refresh**: Strava rotates refresh tokens on every use. The server handles this automatically in memory. If the server restarts and the `.env` token has been rotated, re-run the token helper.
- **Rate limits**: Strava allows 200 requests per 15 minutes and 2,000 per day. Normal conversational use won't come close to these limits.
- **Scopes**: The token helper requests `read`, `activity:read_all`, and `profile:read_all`.
