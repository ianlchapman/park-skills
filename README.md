# park-skills

A collection of Claude Code skills that give AI agents up-to-date knowledge about theme parks — crowd forecasts, opening hours, and weather.

## Skills

- [`park-info`](skills/park-info) — per-park crowd calendar, opening hours, and monthly weather.
- [`park-tickets`](skills/park-tickets) — ticket, pass, and add-on pricing (1-day, multi-day, season/annual passes, fast-track, parking, age-based pricing) for one or more parks.
- [`park-trip-plan`](skills/park-trip-plan) — plan a full trip to a single park: dates, flights, hotel, and itinerary.
- [`multi-park-trip-plan`](skills/multi-park-trip-plan) — plan a multi-park/multi-destination trip: routing, dates per stop, flights/transfers, hotels, and itinerary.

## Data

`skills/park-info/data/*.json` is generated data, one file per park, updated daily by a
[GitHub Action](https://github.com/ianlchapman/park-skills.data/actions) in the
[park-skills.data](https://github.com/ianlchapman/park-skills.data) repo. Don't hand-edit it — changes
will be overwritten on the next run.

`skills/park-tickets/data/*.json` is generated from a hand-curated CSV
(`reference/park_tickets.csv` in park-skills.data) by the same daily pipeline. Ticket pricing is
indicative and only as fresh as its last manual review — check each file's `validity.lastChecked`
field. Don't hand-edit the JSON; update the source CSV instead.

## License

MIT — see [LICENSE](LICENSE).
