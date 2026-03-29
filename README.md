# ⚔️ Character Clash

A real-time multiplayer fantasy battle game for families. Pick your characters, build your team, and let the AI narrator decide who wins.

**Live at:** [marakding.github.io/character-clash](https://marakding.github.io/character-clash)

---

## How to Play

1. **Host** opens the app, enters their name, picks a difficulty, and gets a room code
2. **Other players** open the same URL, enter their name, tap Join, and type the room code
3. Each player **picks one category** from the shuffled list (e.g. Dragon, Shadow Assassin, Storm Caller)
4. Everyone **fills in one character per category** — from any movie, game, anime, myth, or your imagination
5. The **AI narrates an epic battle** between all teams and declares a winner
6. A **Fit Scorecard** reveals how well each character matched their category — bad fits fight at a disadvantage!

---

## Difficulty Levels

| Level | What to expect |
|-------|----------------|
| 🟢 Easy | Broad familiar categories — Dragon, Wizard, Superhero |
| 🟡 Medium | More specific — Trickster God, Storm Caller, Soul Reaper |
| 🔴 Hard | Very niche — Forgotten Deity, Void Entity, Insect Swarm Intelligence |

The harder the difficulty, the more specific the categories — and the harder it is to find a good character fit.

---

## Tech Stack

- **Frontend:** Single HTML file, vanilla JS, hosted on GitHub Pages
- **Realtime multiplayer:** Firebase Realtime Database (free tier)
- **AI narration & scoring:** Anthropic Claude via a Cloudflare Worker proxy
- **No install, no accounts** — just open the URL and play

---

## Architecture

```
Browser (GitHub Pages)
    │
    ├── Firebase Realtime DB  ← syncs game state across all players
    │
    └── Cloudflare Worker     ← proxies Anthropic API (keeps key secret)
            │
            └── Anthropic Claude API  ← generates narration + scorecard
```

---

## Setup (if you want to run your own version)

### 1. Firebase
- Create a project at [firebase.google.com](https://firebase.google.com)
- Enable **Realtime Database** in test mode
- Copy your `firebaseConfig` into `index.html`

### 2. Cloudflare Worker
- Create a free account at [cloudflare.com](https://cloudflare.com)
- Create a new Worker and paste the proxy code:

```javascript
export default {
  async fetch(request, env) {
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'POST, OPTIONS',
          'Access-Control-Allow-Headers': 'Content-Type',
        }
      });
    }
    const body = await request.json();
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': env.ANTHROPIC_KEY,
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify(body)
    });
    const data = await response.json();
    return new Response(JSON.stringify(data), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
      }
    });
  }
}
```

- Add your Anthropic API key as a secret named `ANTHROPIC_KEY`
- Update the Worker URL in `index.html`

### 3. Anthropic API
- Get an API key at [console.anthropic.com](https://console.anthropic.com)
- Add $5 credit — enough for hundreds of games

### 4. GitHub Pages
- Push `index.html` to a public repo
- Enable Pages under Settings → Pages → Deploy from branch → main

---

## Game Design Notes

- **No player limit** — works with 2, 3, 4 or more players
- **Category pool** — 40 categories per difficulty level, shuffled fresh every game
- **Suitability scoring** — the AI judges each character's fit to their category. Perfect fit = full power. Terrible fit = disadvantage (and comic commentary)
- **Scorecard** — after every battle, a card shows each character rated ✅ ⚠️ ❌ with a one-liner

---

## Built with

- Claude (Anthropic) — AI narration and fit scoring
- Firebase — realtime game state
- Cloudflare Workers — API proxy
- GitHub Pages — hosting

---

*Made for family game nights. Best played with strong opinions about who would win in a fight.*
