---
name: park-info
description: Look up theme park data — crowd levels, opening hours, weather, nearest airports, and on-site hotels — for a specific park by name. Use when the user asks about how busy a park will be, when it's open, what the weather/climate is like, which airport to fly into, whether there's a hotel on-site, or wants help picking a date to visit.
---

# park-info

Per-park data for 141 theme parks worldwide: daily crowd predictions and
opening hours (next ~12 months), historical monthly weather averages,
nearest airports, and on-site resort hotels.

## Data location

One JSON file per park in `data/`, named by slug:
`data/<slug>.json` (e.g. `data/alton-towers.json`).

Generated daily by an external pipeline. Don't hand-edit these files.

## Finding the right file

1. Match the user's park name to a slug. Slugs are kebab-case versions of
   park names (e.g. "Alton Towers" → `alton-towers`, "Animal Kingdom" →
   `animal-kingdom`).
2. If unsure of the exact slug, list `data/` and grep/search for a close
   match — don't guess and read a nonexistent file.
3. If the user names a company or region instead of a park ("a Merlin park
   in the UK"), you may need to grep multiple files for `company` /
   `countryIso` / `regionCode` to find candidates.

## File shape

```json
{
  "id": 1,
  "name": "Alton Towers",
  "slug": "alton-towers",
  "company": "Merlin Entertainments",
  "countryIso": "GB",
  "regionCode": "GB-ENG",
  "latitude": 52.9874651,
  "longitude": -1.8864769,
  "tier": "regional",
  "rankedAirports": [],
  "onSiteHotels": [],
  "crowdCalendar": "date,isOpen,openingTime,closingTime,crowdPrediction\n2026-09-25,1,10:00,16:00,30.8\n...",
  "monthlyWeather": [
    { "month": "01", "avgTemp": 3.5, "avgRainfall": 75.3, "avgSunshineHours": 3.9 }
  ]
}
```

- `latitude` / `longitude`: decimal degrees.
- `tier`: park size/significance classification (e.g. `regional`).
- `rankedAirports`: nearest airports, best-served first (may be empty).
- `onSiteHotels`: official on-site/resort hotel names for the park (empty if
  none, or if not yet researched — absence doesn't confirm there's no hotel).

### `crowdCalendar`

CSV string (not an array — kept as CSV to stay token-efficient over ~365
rows), header + one row per day, covering roughly the next 12 months from
generation date:

| Column | Meaning |
|---|---|
| `date` | `YYYY-MM-DD` |
| `isOpen` | `1` open, `0` closed. When `0`, opening/closing times are blank |
| `openingTime` / `closingTime` | 24h `HH:MM`, local to the park |
| `crowdPrediction` | 0–100 relative crowd score for that day. Higher = busier. Not a percentage of capacity — compare days to each other, not to an absolute scale |

To answer questions, parse this CSV (split on `\n`, then `,`) rather than
treating it as JSON. Only look at the date range the user cares about —
don't dump the whole calendar back to the user unless asked.

### `monthlyWeather`

Array of 12 entries (`month` = `"01"`–`"12"`), historical averages:

| Field | Meaning |
|---|---|
| `avgTemp` | Average temperature, °C |
| `avgRainfall` | Average rainfall, mm |
| `avgSunshineHours` | Average sunshine, hours/day |

These are climate normals, not a forecast for a specific date — use them to
answer "what's the weather usually like in [park] in [month]?", not "what
will the weather be on [specific date]".

## Answering common questions

- **"How busy will X be on [date]?"** Read `data/<slug>.json`, find the row
  in `crowdCalendar` matching that date, report `crowdPrediction` and
  whether it's open. Give context (e.g. "that's on the lower end for
  October") by comparing to nearby days/the month, not just the raw number.
- **"Best time to visit X?"** Scan `crowdCalendar` for the date range in
  question, pick low-`crowdPrediction` open days, cross-reference
  `monthlyWeather` for that month if weather matters to the user.
- **"What's X like in weather terms in [month]?"** Use `monthlyWeather`
  for that month.
- **Comparing multiple parks**: read each park's file, compare the same
  field across them (don't mix crowd score with weather when ranking
  unless asked to combine both).
- **"Which airport should I fly into for X?"** Use `rankedAirports` (nearest
  first). If it's empty, say airport data isn't available for that park
  rather than guessing one.
- **"Can I stay on-site at X?" / "What hotels are at X?"** Use `onSiteHotels`.
  An empty list doesn't necessarily mean there's no on-site hotel — some
  parks haven't been researched yet — so say you don't have that
  information rather than stating there's definitely no on-site hotel.

If a requested park has no matching file in `data/`, say so rather than
guessing values.
