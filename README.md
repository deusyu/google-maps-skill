# google-maps-skill

English | [中文](README.zh.md)

AI-powered Google Maps skill for Claude Code — plan international trips through natural conversation.

## Why this skill?

Google Maps is the most widely used map in the world. While restricted in a few countries (China, North Korea), it remains the go-to map for international travel.

**The pain point**: When traveling abroad, Google Maps shows local scripts you can't read — Korean looks like paperclips scattered on the screen, Japanese place names get translated inconsistently between Google Maps and Chinese travel communities (Xiaohongshu, Mafengwo, etc.).

**The solution**: Install this skill in Claude Code, and you get an AI-powered conversation layer on top of real Google Maps data. Ask in plain Chinese (or any language), get structured results with translated place names, route comparisons, and local context — no more squinting at unfamiliar scripts on your phone.

## Features

| Command | What it does |
|---|---|
| `geocode` | Address → coordinates ("Where exactly is Tokyo Tower?") |
| `reverse-geocode` | Coordinates → address ("What's at this pin?") |
| `directions` | Route planning with 4 travel modes: drive, walk, bike, transit |
| `places-search` | Find places by text query ("ramen near Shinjuku") |
| `places-nearby` | Discover places around a location ("cafes within 500m") |
| `place-detail` | Get full details of a specific place |
| `elevation` | Elevation data for any location |
| `timezone` | Timezone info for any location |

## Setup

### Prerequisites

- [Bun](https://bun.sh/) runtime
- [Google Cloud CLI](https://cloud.google.com/sdk/docs/install) (`gcloud`)

### Enable GCP APIs (AI-assisted, 4 commands)

Skip the Google Cloud Console UI. Use `gcloud` CLI instead — paste these commands and you're done.

**Step 1: Login and create project**

```bash
gcloud auth login
gcloud projects create my-maps-skill --name="my-maps-skill"
```

**Step 2: Link billing**

```bash
# Find your billing account ID
gcloud billing accounts list

# Link it
gcloud billing projects link my-maps-skill --billing-account=YOUR_ACCOUNT_ID
```

**Step 3: Enable all 5 APIs at once**

```bash
gcloud services enable \
  geocoding-backend.googleapis.com \
  routes.googleapis.com \
  places.googleapis.com \
  elevation-backend.googleapis.com \
  timezone-backend.googleapis.com \
  --project=my-maps-skill
```

**Step 4: Create API key**

```bash
gcloud services api-keys create --display-name="maps-skill-key" --project=my-maps-skill
```

Save the `keyString` from the output.

### Set environment variable

```bash
export GOOGLE_MAPS_API_KEY="your_key_here"
```

Add to your `~/.zshrc` or `~/.bashrc` to persist.

## Installation

### Option 1: Claude Code Marketplace (Recommended)

```
/plugin marketplace add deusyu/google-maps-skill
/plugin install gmaps@google-maps-skill
```

### Option 2: npx

```bash
npx skills add deusyu/google-maps-skill
```

### Option 3: Manual

```bash
git clone https://github.com/deusyu/google-maps-skill.git
cp -r google-maps-skill/skills/gmaps ~/.claude/skills/
```

## Usage

After installation, just talk to Claude in any language:

```
鹿儿岛核心区有哪些酒店？
```

```
从福冈到�的鹿儿岛怎么走，有几种方式？
```

```
Tokyo Tower 的经纬度是多少？
```

```
Find restaurants near Shibuya Station
```

```
What timezone is Bangkok in?
```

Claude reads the map data, translates place names, and gives you a clear answer — no need to memorize any commands.

## License

[MIT](./LICENSE)
