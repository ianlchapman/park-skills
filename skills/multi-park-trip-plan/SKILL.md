---
name: multi-park-trip-plan
description: Plan a multi-park or multi-destination theme park trip — routing, dates per stop, flights/transfers, hotels, and a day-by-day itinerary — using crowd/weather data and general trip-planning judgement. Use when the user wants to visit more than one park in a single trip (e.g. an Orlando multi-park trip, or a European park-hopping tour), as opposed to a single-park trip (that's park-trip-plan) or a plain data lookup (that's park-info).
---

# multi-park-trip-plan

Plans trips covering multiple parks — whether they're clustered in one
destination (e.g. several Orlando parks) or spread across a route (e.g.
Efteling → Phantasialand → Europa-Park). Builds on the same building blocks
as `park-trip-plan`, applied per stop, plus routing/sequencing across stops.

Use **park-info** for all per-park data. Use whatever flight/hotel/transfer
search skills or tools are available for live pricing and availability —
don't invent fares, hotel rates, or availability. For a trip that turns out
to only involve one park, use `park-trip-plan` instead.

## Required inputs

Ask if not given (reasonable assumptions are fine for minor details, but
state them):

1. **Which parks** (2+), or an open brief like "a week of parks in
   Florida"/"a European coaster tour" — in the latter case, propose a
   shortlist of parks before building the full plan.
2. **Home location** (city/airport).
3. **Total trip length**, or per-park time budget if they already know how
   long they want at each stop.
4. **Party size/composition.**
5. **Trip shape constraints**: fixed start/end dates, must-visit vs.
   nice-to-have parks, preferred order, budget ceiling, appetite for
   long transfers/driving vs. flying between stops.

## Planning process

1. **Look up every candidate park** with park-info: crowd calendar, hours,
   monthly weather, ranked airports, on-site hotels.
2. **Decide the route/order.**
   - Cluster parks that share an airport/region into one base (single
     hotel, day-trips out) rather than moving hotels unnecessarily.
   - For geographically spread parks, sequence stops to minimize
     backtracking (roughly geographic order), and account for realistic
     transfer times (flight, train, or drive) between them — ask the user
     or use a maps/flight tool for transfer feasibility rather than
     guessing large distances are quick.
   - Respect any user-specified order or must-visit priorities over a
     "purely optimal" route.
3. **Check every stop is actually open before allocating dates.** Some parks
   are seasonal and close for weeks or months at a time — check `isOpen`
   across the whole candidate window for each stop *before* committing to a
   route/order, not after. If a requested park is closed for the entire
   window, say so explicitly and either propose alternative dates (if the
   trip window is flexible) or drop/reorder that stop and tell the user
   why, rather than silently producing a plan with a shut park in it.
4. **Allocate dates per stop.** For each open stop, scan its `crowdCalendar`
   for open, low-`crowdPrediction` days within the overall trip window, and
   cross-check `monthlyWeather`. Where the trip window is fixed, allocate
   days across stops in proportion to what each park needs (bigger parks/
   resorts get more days) and flag if the window is too tight to do all
   requested parks justice.
5. **Pace the route.** Multi-park trips fail when they're packed too tight.
   As a default: no more than 3 consecutive park days before a rest or
   low-key travel day, and any stop involving a long-haul flight or a
   transfer over ~4 hours gets a light arrival day rather than a park day
   immediately after. Deviate from this only if the user explicitly asks
   for a packed schedule, and say so if you do.
6. **Flights/transfers.** Use `rankedAirports` per stop plus the user's home
   location for the first/last leg. Search inter-stop transport with an
   available tool (flights, trains, car hire — e.g. existing rental-car or
   flight-search skills) where relevant; if no live tool is available,
   state what should be searched (route, dates) instead of fabricating
   options. For cross-border legs, don't assume transfer times — use a
   maps/flight/rail tool or clearly flag the estimate as unverified.
7. **Hotels per stop.** Prefer `onSiteHotels` where available and it makes
   sense to stay close to that park; otherwise use an available hotel
   search tool for a base convenient to that leg (or to two clustered
   parks). If no hotel tool is available, say so rather than guessing rates.
8. **Day-by-day shape.** Build a single continuous itinerary across the
   whole trip: travel/transfer days, each park's visit day(s) with opening
   hours and suggested arrival, and buffer days for long transfers or rest.
9. **Sanity-check against weather** for each stop's month, and flag anything
   that materially affects the plan (heavy rain, extreme heat, etc.).
10. **Cross-border check.** If stops span more than one country, flag
    currency changes and time zone changes per stop (dates/times in the
    plan should always be stated as local to that stop), and note if travel
    documents (visas, ID requirements) might be relevant — without
    guessing specifics, just flag it for the user to verify.

## Output format

Present the finished plan consistently, in this order:

1. **Summary line** — parks in order, overall dates, party size, one-line
   rationale for the route and date allocation.
2. **Route overview** — ordered list of stops with base location and dates
   (stated local to that stop) at each, so the shape of the trip is clear
   before the detail. Note currency/time-zone changes here if the route
   crosses borders.
3. **Flights/transfers** — inbound, each inter-stop leg, and the return;
   what was found (or what to search if no live tool was available).
4. **Hotels** — one per stop (or per cluster), with reasoning.
5. **Day-by-day plan** — one line/short block per day across the whole trip:
   travel days, each park day with opening hours and suggested arrival,
   rest/buffer days.
6. **Notes/caveats** — weather flags, data gaps, tight-timing warnings,
   anything assumed rather than confirmed.

Keep it scannable — headings and short bullets per stop, not dense prose.
State clearly which parts are confirmed data vs. assumptions or general
advice, so the user knows what still needs booking.

### Template

```
## [Trip name/route] — [overall date range]

[Party size] · [route rationale: why this order/allocation]

### Route
- Stop 1: [Park] — [base location], [dates, local]
- Stop 2: [Park] — [base location], [dates, local] [currency/timezone note if changed]
- ...

### Flights/transfers
- Inbound: [origin] → [stop 1], [date] — [found option or "search: ..."]
- Stop 1 → Stop 2: [mode], [date] — [found option or "search: ..."]
- ...
- Return: [last stop] → [origin], [date] — [found option or "search: ..."]

### Hotels
- Stop 1: [Hotel] ([on-site/off-site]) — [why]
- Stop 2: [Hotel] ([on-site/off-site]) — [why]

### Day by day
- Day 1 ([date]): Travel — arrive [stop 1], [transfer]
- Day 2 ([date]): [Park] — open [hh:mm]–[hh:mm], crowd [level]
- Day 3 ([date]): Travel — [stop 1] → [stop 2]
- ...
- Day N ([date]): Travel home

### Notes
- [Closed-park flags, weather flags, pacing calls, cross-border notes, assumptions]
```

If any named park has no `park-info` data file, say so for that park
specifically (don't drop it from the plan silently), and fall back to
general trip-planning judgement for it while still using real data for the
others.
