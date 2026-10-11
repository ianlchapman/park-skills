---
name: park-trip-plan
description: Plan a full trip to a single theme park — dates, flights, hotel, and park day(s) — using crowd/weather data and general trip-planning judgement. Use when the user wants a trip plan, itinerary, or "help me plan a visit to X" for one park, as opposed to just a data lookup (that's park-info) or a multi-park/multi-destination trip (that's multi-park-trip-plan).
---

# park-trip-plan

Turns a single-park visit into a concrete, bookable plan: when to go, how to
get there, where to stay, and a day-by-day shape — grounded in real data, not
generic travel advice.

This skill orchestrates other tools rather than replacing them. Use
**park-info** for all park data (crowds, hours, weather, airports, on-site
hotels). Use **park-tickets** for ticket/pass pricing for the park. Use
whatever flight/hotel search skills or tools are available (e.g. flight or
hotel search skills) for live pricing and availability — don't invent
fares, hotel rates, or availability.

## Required inputs

Before planning, get (ask if not given, but make reasonable assumptions and
state them rather than blocking on minor details):

1. **Which park.**
2. **Home location** (city/airport) — needed for flights.
3. **Trip length** or fixed dates. If only length is given, you'll recommend
   dates; if fixed dates are given, work within them.
4. **Party size/composition** (adults, kids, ages if relevant) — affects
   ticket/room choices and pacing.
5. **Anything else stated**: budget ceiling, must-avoid dates, weather
   preference, whether they want to stay on-site.

## Planning process

1. **Look up the park** with park-info: crowd calendar, opening hours,
   monthly weather, ranked airports, on-site hotels.
2. **Choose or validate dates.**
   - If the user gave a date range to pick within, scan `crowdCalendar` for
     open days with low `crowdPrediction` in that range, and cross-check
     `monthlyWeather` for the relevant month(s).
   - If the user gave fixed dates, just report what those dates look like
     (crowd level, expected weather, opening hours) so they know what to
     expect — don't silently override their dates.
   - Prefer weekdays over weekends and shoulder-season over peak when it
     doesn't conflict with the user's constraints, and say why.
3. **Flights.** Use `rankedAirports` for the destination side and the user's
   home location for the origin side. Search with an available flight tool
   for the chosen dates; if none is available, describe what to search for
   (route, dates, cabin) rather than fabricating fares.
4. **Tickets.** Look up ticket options with park-tickets: the standard
   1-day ticket price for the visit dates, and whether a multi-day ticket
   or pass is cheaper than buying single days if the trip covers multiple
   park days. Flag add-ons (fast-track, parking) only if the user mentions
   wanting to skip lines or is driving.
5. **Hotel.** Check `onSiteHotels` first — an on-site stay is usually the
   right default recommendation for a single-park trip (shorter transfers,
   early park access if the park offers it). If none exists or the user
   wants alternatives, use an available hotel search tool for nearby
   options. If no hotel tool is available, say so rather than guessing rates.
6. **Day-by-day shape.** For the park day(s): note opening/closing times from
   the crowd calendar, and suggest arrival time (before opening on
   high-crowd days, more relaxed on low-crowd days). Leave travel days light.
7. **Sanity-check against weather, and build in contingency.** If
   `monthlyWeather` shows high rainfall or extreme temperature for the
   chosen month, flag it and suggest packing/planning implications — don't
   just omit it because it's inconvenient. For a multi-day trip in a
   higher-rainfall month, prefer allocating an extra park day (or a
   flexible/unstructured day) over a single tightly-packed park day, so one
   washed-out day doesn't sink the whole trip. Say explicitly which day (if
   any) is the rain-day buffer.
8. **Ground transport.** Don't jump straight from flights to hotel — include
   the airport-to-hotel/park leg (rental car, shuttle, taxi, transit) using
   an available tool where possible, or state what to arrange if none is
   available. Factor realistic transfer time into the arrival day's
   schedule (e.g. don't plan a same-day park visit right after a long-haul
   flight plus a long transfer).

## Output format

Present the finished plan consistently, in this order:

1. **Summary line** — park, dates, party size, one-line rationale for the
   dates chosen (or a note on the given dates' crowd/weather).
2. **Flights** — outbound/return, airport(s), and what was found (or what to
   search if no live tool was available).
3. **Tickets** — ticket type recommended (1-day vs. multi-day vs. pass),
   price per person, and any relevant add-ons.
4. **Hotel** — recommendation with reasoning (on-site vs. off-site), and
   what was found.
5. **Day-by-day plan** — one line/short block per day: travel days, park
   day(s) with opening hours and suggested arrival, any rest/buffer days.
6. **Notes/caveats** — weather flags, data gaps (e.g. "no on-site hotel data
   for this park"), anything assumed rather than confirmed.

Keep it scannable — headings and short bullets, not dense prose. State
clearly which parts are confirmed data (from park-info or a live search) vs.
assumptions or general advice, so the user knows what still needs booking.

### Template

```
## [Park name] trip — [date range]

[Party size] · [rationale for these dates: crowd/weather one-liner]

### Flights
- Out: [origin] → [destination airport], [date] — [found fare/flight or "search: ..."]
- Return: [destination airport] → [origin], [date] — [found fare/flight or "search: ..."]

### Getting there
- [Airport] → [hotel/park]: [mode, est. time] — [found option or "search: ..."]

### Tickets
- [Ticket type] — [price per person], [multi-day/pass note if cheaper], [add-ons if relevant]

### Hotel
- [Hotel name] ([on-site/off-site]) — [why], [found rate or "search: ..."]

### Day by day
- Day 1 ([date]): Travel — arrive [time], [transfer]
- Day 2 ([date]): [Park] — open [hh:mm]–[hh:mm], arrive [suggested time], crowd [level]
- Day 3 ([date]): Buffer/rain-day — [what this covers]
- Day N ([date]): Travel home — depart [time]

### Notes
- [Weather flag, data gaps, assumptions]
```

If the named park has no `park-info` data file, say so and fall back to
general trip-planning judgement, flagging that crowd/weather/hotel specifics
aren't available for that park.
