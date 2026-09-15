# The Coffee App — Concept

Working document describing what we are building. `MANIFESTO.md` defines the constraints; this defines the product. It will evolve.

## Thesis

Existing apps each cover one slice: café maps (Roasters), bag reviews (Roastguide), brew logging (Beanconqueror). None connects them through a personal taste profile. Coffunity tried to be the Untappd of coffee (SCA best new product 2018, seed funded, 150K+ coffees in its database) and is now defunct; its data died with the company. Open data is how we avoid repeating that. Our thesis: a café review, a coffee bag, and a brew recipe should all feed one model of what *this person* enjoys, and that model should drive every recommendation. Everything else is a feature; the taste graph is the product.

## Core entities

The recommendation engine only works if experiences attach to canonical things. The entity graph is the primary design artifact.

- **Origin / Farm** — country, region, farm or cooperative, altitude, variety. Sparse and wiki-like; filled in over time.
- **Roaster** — name, location, website, subscribable.
- **Coffee** — a specific offering from a roaster: origin(s), process, variety, roast level, harvest/lot, roast date range. The hardest entity: new lots every few months, no barcode standard, same farm under different names. Needs strong search-and-match on creation, dedup tooling, and unverified status until confirmed.
- **Café** — name, coordinates, hours, subscribable. Serves a list of Coffees (the Café→Coffee edge, editable by the café or by users; this is what makes "bags available near you" answerable). Amenities as a fixed set of structured tags (wifi, laptop-friendly, outlets, food, outdoor seating), OSM-style. No full food menus.
- **Experience** (diary entry) — the atomic unit. Links to a Coffee and optionally a Café. Rating, canonical descriptors, free descriptors, optional text, photo, and brew recipe. Private by default; user may share.
- **Recipe** — attached to an experience: method, dose, water amount and type, temperature, grind, time, steps.
- **Review** — a shared experience of a Café or Coffee. Personal expression; excluded from open data exports.
- **Event** — hosted by a café, roaster, or person: cuppings, workshops, throwdowns. Subscribers are notified; people can RSVP.
- **Post** — coffee-related writing by a user: trip reports, comparisons, opinions, tips. May attach to entities. Discovered via search, entity pages, and author subscriptions, never via an algorithmic feed.
- **User** — account, taste profile, reputation, subscriptions. Personal data, never exported.
- **Wiki article** — CC BY-SA content about origins, processing, roasting, brewing, gear.

Every record carries provenance: created by (user id), source, license, verification status.

## Taste model

- **Onboarding**: a very short quiz to bootstrap (roast preference, black vs. milk, pick a few descriptors). Nothing more; the diary does the real work.
- **Descriptors**: two layers. Canonical vocabulary (structured, mapped to a flavor-wheel-style hierarchy) for stable, explainable comparison. Free descriptors kept verbatim, embedded, and used as signal; promoted or mapped to canonical as usage grows.
- **Taste vector**: derived per user from ratings × descriptors across experiences, plus embeddings of free text and private diary notes. Recomputed incrementally. Personal data is analyzed only to serve its owner; recommendations should be able to show why.
- **Recommendations**:
  - Similar-taste review surfacing: rank a café's or coffee's reviews by similarity between reader's and author's vectors. Cheap, no LLM.
  - Coffee/café suggestions: nearest neighbors in taste space, filtered by availability (local roasters, cafés nearby, bags stocked locally).
  - Experiments: items at a controlled distance from the user's vector, always labeled as such. Feedback on experiments maps the user's boundaries.
  - AI summaries: not per user. Cluster users into a small number of taste archetypes, generate one cached summary per café/coffee per archetype, regenerate when enough new reviews arrive. Cost is constant per entity.
  - Collaborative filtering later, once user overlap is sufficient.

## Identifiers

There is no consumer-facing standard for identifying a specific coffee lot (GTIN identifies a product, not a harvest; wine has LWIN, coffee has nothing). We create one, and our own database is its registry.

- Every Coffee gets a stable, permanent, human-readable public ID that resolves to a URL (e.g. `<domain>/c/7Q3K9M`). Roasters, cafés, and farms get IDs too.
- The ID scheme and resolution API are published under an open license (spec CC0, registry data ODbL) so anyone can generate, resolve, and mirror IDs. A standard, not a lock-in.
- Roasters print the ID as a QR code on bags. Scanning shows origin, process, harvest, reviews from similar-taste users, and recipes, with one-tap "log this." Roasters get a page they control, analytics across taste archetypes, and a channel to announce the next lot to everyone who logged this one.
- Every scanned bag is a diary entry attached to the right entity: no search-and-match, and canonical by construction. Coffees without a printed ID still get one on creation, so there is no dependency on roaster adoption.
- Adoption: a few local roasters first (this pilot is inherently local); the coffee page must be genuinely useful before pitching. Name the scheme early; once printed it is permanent.

## Map

- OpenStreetMap data, rendered with MapLibre GL. Tiles from a provider (MapTiler, Stadia) or self-hosted (Protomaps).
- Map behind an adapter interface (render pins, cluster, pan/zoom, geocode) so the provider can be swapped.
- Coordinates and place data stored in our own DB; provider IDs are never primary keys.
- If cafés are seeded from OSM, ODbL share-alike applies; our data is ODbL anyway, so this is compatible.

## Contribution and moderation

Two paths into the database:

- **UI path** (everyday users): add or edit a café, coffee, roaster. New records are "unverified" until confirmed by another user or approved by a trusted contributor. Process to be designed; trusted status derives from reputation.
- **Import path** (bulk contributors): proposal in the repository with dataset, source, and license. Maintainer checks license compatibility (no Google Maps, no competitor data). Approved data is imported by script, attributed to the contributor's account, and marked unverified until touched by UI users. Later: an API with an import scope for trusted contributors.

Descriptor curation: free descriptors reviewed periodically; frequent ones promoted to canonical or mapped to a canonical parent.

## Reputation

Grows from usefulness: records you added get confirmed and used, reviews get marked useful, edits get accepted, recipes get followed. Never from volume, streaks, or logins. Authors see when their contributions helped someone ("was this useful?" rather than likes; batched notifications; never ranked by raw count). Reputation unlocks trusted-contributor capabilities.

## Subscriptions and events

Users subscribe to roasters, cafés, and hosts for new releases and events. Events support RSVP so hosts and attendees know attendance. No activity feed, no follower counts as status.

## Wiki

Structured entity pages first (origin, variety, process, roaster) rather than prose articles; prose wiki grows on top. Recipes surface on coffee and method pages.

## Open data

Periodic export of reference data (roasters, coffees, cafés, farms, aggregates) under ODbL, plus a public read API. Reviews and all personal data excluded. Users can export their own data.

## Revenue (eventual, not a current goal)

Options consistent with the manifesto, roughly in order of fit:
1. Tools for roasters and cafés: claim a page, announce releases, see how coffees land across taste archetypes, events, QR/ID printing and analytics; possibly POS/café management later. Code open; the hosted service is what's paid for.
2. Commerce: tracked "buy from roaster" links first; a marketplace only once demand data exists.
3. Gear affiliate links from the wiki.
Never: ads, paid placement, data sales, equity investment.

## MVP wedge (proposed)

Coffee (bag) diary + roaster/coffee database + taste-based recommendations. Cafés enter as "where I drank it" and grow into the map. Recipes piggyback on diary entries. Wiki and events follow.

Go-to-market: seed a community, not a city. Recruit early users from specialty coffee forums and communities wherever they are; coffees and roasters are not geographically bound, and taste data from anywhere improves recommendations for everyone. Café density emerges region by region as contributors map their own cities, as OSM grew. The roaster/QR pilot starts locally: convincing the first roasters to print an ID with no traction to show is relationship-driven, and that is easiest in person. Once a handful have adopted it, they become the proof for approaching roasters anywhere. Consequence: the data model is international from day one (multi-script names, units, currencies if prices ever appear).

Rationale: this is where the taste graph starts, where commerce eventually lives, and the least well-served corner. The identifier scheme makes it stronger: bags carrying our QR are the acquisition channel. A café-first wedge competes head-on with Roasters and needs the taste data anyway.

## Open questions

- Which forums and communities to seed from, and which local roasters to pilot with.
- Wedge: bag-first (proposed) or café-first.
- Platform at launch: mobile, web, or both. This decides more of the stack than anything else.
- Moderation process and what "trusted contributor" concretely means.
- Whether public reviews are included in exports as aggregates only, or not at all.
- Roaster validation: talk to a few roasters before building B2B tools, including whether they would print a QR/ID on bags.
- Name for the identifier scheme.

## Builder context

Solo developer, building with AI assistance, no funding. Stack is decided in a separate session using this document. Constraints that follow from the manifesto and the situation:

- **Self-hostable by anyone.** Standard, portable components (e.g. plain Postgres, containerized deployment); no logic locked into proprietary vendor features. Whether the flagship instance runs on managed or self-operated infrastructure is a separate choice, made on cost and operational load, not principle.
- **Contributor-friendly.** One-command local setup; tools with enough adoption that new contributors and AI assistants are already fluent in them.
- **Stable and well-maintained**, but not necessarily conventional. Excellent tools with real momentum are preferred over defaults chosen for familiarity; tools with a single maintainer or an uncertain future are avoided.
- **Permanent public identifiers** (see Identifiers) mean URL structure and ID generation are decided once.
