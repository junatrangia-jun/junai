# Jun — your executive assistant

Jun reads your Google Calendar and Gmail, blocks the days you're unavailable, tells you when to leave for in-person meetings (with live traffic from Google Maps), recommends going online when the trip isn't worth it, moves meetings, finds restaurants and places, and sorts your inbox. Talk to her or type.

Built with **Next.js 15 + TypeScript** (no Python), so it deploys to Vercel as-is.

## What's inside

| Area | Where |
| --- | --- |
| Jun's personality (edit her voice here) | `src/lib/persona.ts` |
| Jun's brain: tools + agent loop (Claude) | `src/lib/jun.ts` |
| "Should this be online?" rules | `src/lib/advisor.ts` |
| Travel times + places (Google Maps) | `src/lib/maps.ts` |
| Calendar + Gmail | `src/lib/google.ts` |
| Inbox triage | `src/lib/triage.ts` |
| Voice in/out | `src/hooks/useVoice.ts`, `src/app/api/voice/route.ts` |
| Reminders + alerts | `src/components/AppContext.tsx` |
| Screens | `src/components/*View.tsx`, `JunPanel.tsx` |

---

## 1. Put it on GitHub

```bash
cd jun
git init
git add .
git commit -m "Hello, Jun"
# create an empty repo on github.com (no README), then:
git remote add origin https://github.com/YOUR-USERNAME/jun.git
git branch -M main
git push -u origin main
```

## 2. Get your keys (about 20 minutes, once)

### Google sign-in, Calendar and Gmail
1. Go to https://console.cloud.google.com and create a project (e.g. "Jun").
2. **APIs & Services → Library**: enable **Google Calendar API** and **Gmail API**.
3. **APIs & Services → OAuth consent screen**: choose *External*, fill in the app name and your email. Under **Test users**, add your own Gmail address. (While the app is in Testing mode, only test users can sign in, which is exactly what you want for a personal assistant.)
4. **APIs & Services → Credentials → Create credentials → OAuth client ID** → *Web application*.
   - Authorized redirect URIs: add both
     - `http://localhost:3000/api/auth/callback/google`
     - `https://YOUR-APP.vercel.app/api/auth/callback/google` (add this after step 3 below, once you know your Vercel URL)
5. Copy the **Client ID** and **Client secret**.

### Google Maps (travel time + restaurants)
1. Same Google Cloud project → **Library**: enable **Routes API** and **Places API (New)**.
2. **Credentials → Create credentials → API key**. Click it and restrict it to those two APIs.
3. Google Maps needs billing enabled on the project; personal use typically stays within the monthly free credit, but set a budget alert to be safe.

### Jun's brain (Claude)
Create an API key at https://console.anthropic.com.

### Jun's voice (optional but recommended)
Without this, Jun uses your browser's built-in voice. For her own voice:
1. Sign up at https://elevenlabs.io, open **Voices → Voice Design**, and describe her, e.g.
   *"Warm, soft-spoken woman in her early 30s, calm and confident, a hint of playfulness, clear diction, speaks with quiet authority."*
2. Save the voice, copy its **Voice ID**, and create an API key.

## 3. Deploy on Vercel

1. Go to https://vercel.com/new and import your `jun` GitHub repo. Framework is detected as Next.js; leave the defaults.
2. Before clicking Deploy, open **Environment Variables** and add everything from `.env.example`:

| Name | Value |
| --- | --- |
| `GOOGLE_CLIENT_ID` | from Google |
| `GOOGLE_CLIENT_SECRET` | from Google |
| `NEXTAUTH_SECRET` | run `openssl rand -base64 32` and paste the result |
| `NEXTAUTH_URL` | `https://YOUR-APP.vercel.app` |
| `GOOGLE_MAPS_API_KEY` | from Google |
| `ANTHROPIC_API_KEY` | from Anthropic |
| `ANTHROPIC_MODEL` | `claude-sonnet-5-5` |
| `ANTHROPIC_FAST_MODEL` | `claude-haiku-5-5` |
| `ELEVENLABS_API_KEY` | optional |
| `ELEVENLABS_VOICE_ID` | optional |

3. Deploy. Copy your production URL, set `NEXTAUTH_URL` to it, add the redirect URI from step 2.4 in Google Cloud, then **Redeploy** in Vercel.
4. Open the app, sign in with Google, click your avatar → **Settings** and add your **home base** so Jun can time your trips. Turn on **desktop alerts** while you're there.

Every push to `main` redeploys automatically.

## Run it locally

```bash
cp .env.example .env.local   # fill in the values
npm install
npm run dev                  # http://localhost:3000
```

---

## Using Jun

- **Today**: Jun's brief at the top (press *Hear it from Jun*), then your day as a timeline. In-person meetings show a hatched travel band sized to the trip, a *leave by* time, and a verdict: *Go in person*, *Consider online* or *Take it online*. One tap switches a meeting to Google Meet or moves it.
- **Availability**: tap days on the calendar (first day, then last day) and block them. They show as busy in Google Calendar.
- **Inbox**: last week's mail sorted into *Needs you*, *Can wait* and *For your info*, with suggested replies saved as Gmail drafts.
- **Talk to Jun**: the mic button (Chrome, Edge, Safari) or type. Try:
  - "What does my day look like?"
  - "Move my 3pm to tomorrow afternoon."
  - "How long to get to my 2pm, and should I just go online?"
  - "Find a quiet Japanese place for lunch near my 1pm."
  - "Block December 22 to January 2, holiday."
  - "Anything urgent in my inbox? Draft a reply to the first one."

Jun always asks before she moves a meeting, switches it online or blocks dates. Emails are only saved as drafts, never sent.

### How "should I go online?" works
From `src/lib/advisor.ts`:
- If travel + your buffer is longer than the free time before the meeting → **Take it online**
- If the round trip is 1.5× the meeting length or more → **Take it online**
- If the round trip is longer than the meeting, or it's 45+ minutes each way → **Consider online**
- Otherwise → **Go in person**

Adjust the numbers to taste.

### Alerts
While Jun is open in a browser tab (or installed as an app, see below), she alerts you before you need to leave and again when it's time to go, with a spoken heads-up if voice is on. Install her on your phone or desktop with your browser's **Install app** / **Add to Home Screen** option.

## Good next steps
- **Alerts when the app is closed**: add Web Push (store push subscriptions in Vercel KV / Upstash) plus a Vercel Cron job that checks upcoming meetings every few minutes.
- **Morning email brief**: a cron route that emails you Jun's brief at 7am.
- **Smarter inbox**: let Jun auto-label and archive *For your info* mail once you trust her sorting.

## Privacy
Google tokens live in an encrypted, HTTP-only session cookie and are only used server-side. Nothing is stored in a database. Calendar, mail and location details are sent to Anthropic (Jun's reasoning), Google Maps (travel and places) and, if enabled, ElevenLabs (her voice) only when you use those features.
