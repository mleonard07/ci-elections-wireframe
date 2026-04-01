# CI Elections 2026 — Developer Handoff

This wireframe is a single-file prototype (`index.html`) built to demonstrate the Elections 2026 reader experience for Community Impact. It is **fully functional as a demo** but uses stub data and dummy auth that must be replaced before production.

---

## Running locally

```bash
cd ci-elections-wireframe
python3 -m http.server 5050
# open http://localhost:5050
```

No build step. No dependencies to install. Everything is inline in `index.html`.

---

## Integration points

There are four places where the wireframe uses stubs that you will replace with real systems.

### 1. Race data — ERP/CRM API
**Location:** `index.html` ~line 740 (`const RACE_DATA`) and ~line 919 (`const DEFAULT_RACES`)

Race and candidate data is currently hardcoded as a JavaScript object. In production, replace this with an API fetch from CI's ERP/CRM.

**Current shape of a community's data:**
```js
'Round Rock': {
  groups: [
    {
      label: '🏛 US & State',
      races: [
        {
          id: 'rr-us10',
          office: 'US House · District 10 · Republican Primary',
          name: 'US Representative, District 10',
          status: 'winner',          // 'winner' | 'runoff' | 'uncontested' | 'pending'
          candidates: [
            { name: 'Michael McCaul', party: 'r', pct: 72, votes: '38,412', winner: true },
            { name: 'Ted Kraus',      party: 'r', pct: 28, votes: '15,001' },
          ],
          qa: {                       // optional — omit if no Q&A available
            q: 'What is your top priority?',
            answers: [
              { name: 'Michael McCaul', text: 'Border security and fiscal responsibility.' },
            ]
          }
        }
      ]
    }
  ]
}
```

`DEFAULT_RACES` (~line 919) is the fallback shown for communities with no data yet. Remove it once the API covers all communities.

The filter pills map to group labels — make sure your API returns the same label strings:
- `'🏛 US & State'`
- `'🎓 School Board'`
- `'🏙 City Council'`
- `'🏛 County'`
- `'📋 Propositions'`

---

### 2. Authentication
**Location:** `index.html` ~line 1347

Two items to replace:

**a) `DUMMY_USERS` array** — remove entirely. This is only for demo login.
```js
// DELETE THIS before production
const DUMMY_USERS = [
  { email: 'admin@communityimpact.com', password: 'impact123', ... },
  ...
];
```

**b) `authenticate(email, password)` function** (~line 1369) — replace the body with a call to CI's existing auth API:
```js
// REPLACE this function body with your real auth integration
async function authenticate(email, password) {
  // example — adapt to your actual auth endpoint:
  const res = await fetch('/api/auth/login', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });
  if (!res.ok) return null;
  return await res.json(); // must return { email, name, initials } or null
}
```

The function must return either `null` (failed) or an object with `{ email, name, initials }`. Everything else in the auth flow (modal, nav state, sidebar state) stays the same.

> Note: `submitLogin()` currently calls `authenticate()` synchronously. If you make it async, update `submitLogin()` to `await authenticate(...)` as well (it's ~10 lines below the function).

---

### 3. Saved races — favorites persistence
**Location:** `index.html` ~line 1384, 1396, 1452

Favorites are currently stored in `localStorage` keyed by user email:
```js
localStorage.getItem('ci_favorites_' + user.email)
localStorage.setItem('ci_favorites_' + user.email, JSON.stringify(favorites))
```

**Shape of the favorites object:**
```js
{
  'rr-us10': { office: 'US House · District 10...', name: 'US Representative...', community: 'Round Rock' },
  'rr-rrisd1': { ... }
}
```

To move this to a backend API, replace the three `localStorage` calls:
- On login (~line 1384): fetch saved races for the user from your API
- On toggle (~line 1452): POST/DELETE to your API
- On sign-out (~line 1396): no change needed (in-memory clear only)

---

### 4. Address geocoding
**Location:** `index.html` ~line 1045

The address search uses [Nominatim](https://nominatim.openstreetmap.org/) (OpenStreetMap's free geocoding API). **This is not suitable for production** — it has rate limits and requires attribution.

```js
// REPLACE with a paid geocoding API before launch
const url = `https://nominatim.openstreetmap.org/search?q=${encodeURIComponent(query)}&format=json&limit=1&countrycodes=us`;
```

Recommended replacements: Google Maps Geocoding API, Mapbox Geocoding, or SmartyStreets. The function expects `results[0].lat` and `results[0].lon` from the response — update the property names to match your chosen API's response format.

---

## What's real vs. placeholder

| Feature | Status |
|---|---|
| Map (Leaflet + CartoDB tiles) | Production-ready |
| Community boundaries | Real coordinates — verify/update as coverage expands |
| Pflugerville–Hutto race data | Real (March 2026 primary results from CI voter guide) |
| Round Rock, Cedar Park race data | Placeholder — replace via ERP/CRM API |
| Auth flow (UI, modal, nav state) | Production-ready UI — replace `authenticate()` only |
| Saved races / favorites (UI) | Production-ready UI — replace localStorage with API |
| Print Ballot Guide | Production-ready |
| Filter pills + Results toggle | Production-ready |
| "Find your polling location" link | Links to Texas SOS voter portal — update for non-TX markets |
| Mobile layout | Production-ready |
| Email/push notifications | Out of scope for phase 1 |
| Candidate profile pages | Not built — cards link to `communityimpact.com/races/[slug]` as placeholder |

---

## Questions

Contact Marie Leonard (VP Audience & Development) for product questions, or refer to the voter guide at `communityimpact.com/voter-guide/` for source data.
