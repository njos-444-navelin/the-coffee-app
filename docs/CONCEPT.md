# The Coffee App — Concept

Working document describing what we are building. `MANIFESTO.md` defines the constraints; this defines the product. It will evolve.

## Thesis

Existing apps each cover one slice: café maps (Roasters), bag reviews (Roastguide), brew logging (Beanconqueror). None connects them through a personal taste profile. Coffunity tried to be the Untappd of coffee (SCA best new product 2018, seed funded, 150K+ coffees in its database) and is now defunct; its data died with the company. Open data is how we avoid repeating that. Our thesis: a café review, a coffee bag, and a brew recipe should all feed one model of what *this person* enjoys, and that model should drive every recommendation. Everything else is a feature; the taste graph is the product.

## Core entities

The recommendation engine only works if experiences attach to canonical things. The entity graph is the primary design artifact.

- **Origin / Farm** — country, region, farm or cooperative, altitude, variety. Sparse and wiki-like; filled in over time.
- **Roaster** — name, location, website, subscribable. Roasting style is never a field anyone fills in: it is learned from how people describe the roaster's coffees (`docs/RECOMMENDATIONS.md`, Attribution). A claimed page may describe its approach in prose, and that prose is text signal like any other.
- **Coffee** — a specific offering from a roaster: origin(s), process, variety, roast level, intended use as printed by the roaster (filter, espresso, omni, for milk), harvest/lot, roast date range. The hardest entity: new lots every few months, no barcode standard, same farm under different names. Needs strong search-and-match on creation, dedup tooling, and unverified status until confirmed.
- **Café** — name, coordinates, hours, subscribable. Serves a list of Coffees (the Café→Coffee edge, editable by the café or by users; this is what makes "bags available near you" answerable). Also serves one or more Roasters (the Café→Roaster edge): cafés change roasters rarely, so this edge stays accurate when the coffee list is stale or empty, and with the roaster's fingerprint it carries most of the taste signal for "cafés whose coffee fits you". Amenities as a fixed set of structured tags (wifi, laptop-friendly, outlets, food, outdoor seating), OSM-style. No full food menus.
- **Experience** (diary entry) — the atomic unit: I drank this coffee, here, now, as this drink. Links to a Coffee, and to a place (home, a Café, elsewhere). The drink (black filter, espresso, a milk drink, or something else, with the specific drink named) is the one field that cannot be skipped: one tap at logging. Without it a verdict cannot be attributed to a pairing, and the system cannot tell a bad cappuccino from a bad coffee. Verdicts are always about a coffee in a drink, never about the coffee alone (`docs/RECOMMENDATIONS.md`, Drinks). Carries two separate, optional verdicts: one on the coffee (rating, canonical descriptors, free descriptors, notes, photo, brew recipe if self-brewed) and, when the place is a café, one on the café (service, space, amenities). The bag in hand carries its roast date and harvest date when known (most specialty bags print at least the roast date); both optional. A self-brewed entry also carries the person's own assessment of the brew: whether it went right, went wrong, or they are not sure. That assessment, not the score, decides how much a cup says about the coffee (`docs/RECOMMENDATIONS.md`, Attribution). Private by default; either verdict may be shared. The diary is one timeline of uniform entries that differ only in their details, never split into sections; filters narrow it by where it was drunk (home, café, elsewhere), by coffee, or by café. Repeated brews of one bag group into a view of that bag.
- **Recipe** — attached to an experience: method, dose, water amount and type, temperature, grind, time, steps, and any ingredients beyond water (milk, tonic, syrup). Coffee cocktails and milk drinks are recipes like any other; the taste model never needs to understand tonic, only which coffee went in and what the drink was.
- **Review** — the shared verdict of an experience, on a Café or on a Coffee. Not a separate thing the user writes; a café visit becomes a café review when its café verdict is filled in and shared. Personal expression; excluded from open data exports.
- **Event** — hosted by a café, roaster, or person: cuppings, workshops, throwdowns. Subscribers are notified; people can RSVP.
- **Post** — coffee-related writing by a user: trip reports, comparisons, opinions, tips. May attach to entities. Discovered via search, entity pages, and author subscriptions, never via an algorithmic feed.
- **User** — account, taste profile, reputation, subscriptions. Personal data, never exported.
- **Wiki article** — CC BY-SA content about origins, processing, roasting, brewing, gear.

Every record carries provenance: created by (user id), source, license, verification status.

## Taste model

- **Onboarding**: a very short quiz to bootstrap (roast preference, black vs. milk, pick a few descriptors). Nothing more; the diary does the real work.
- **Descriptors**: two layers. Canonical vocabulary (structured, mapped to a flavor-wheel-style hierarchy) for stable, explainable comparison. Free descriptors kept verbatim, embedded, and used as signal; promoted or mapped to canonical as usage grows.
- **Taste vector**: derived per user from ratings × descriptors across experiences, plus embeddings of free text and private diary notes. Recomputed incrementally. Personal data is analyzed only to serve its owner; recommendations should be able to show why.
- **A person is a set of points, not an average**: taste is plural. Someone may love heavily fermented coffees from anywhere, clean washed Ethiopians, and every natural Kenyan; averaging those into one vector describes nobody. Loved and disliked entities are both kept as individual anchors; recommendation is attraction to the loved ones minus repulsion from the disliked ones. A clustering method that discovers the number of clusters itself groups anchors into named tastes for explanation. One taste or twenty, both are fine. Nobody is asked to state rules about their own taste; a note is read into observations about that cup ("too fermented for me"), the person confirms only what was read, and conditional preferences emerge from where the points sit. Curiosity is separate from liking: the person can declare what they are curious about (new processes, varieties, roast styles, brew methods) and what they are not, and which experiments they take up refines it. Cafés work the same way: a counter-culture community spot and a posh high-end one can both be genuine tastes of one person.
- **A cup is evidence about several things**: the café, the coffee, the roaster, the bag's freshness, and the person's own brewing. How much it says about each depends on where it was drunk and what the person said about it, never on the score alone. At home the person controlled the brew, so a bad cup speaks about the brewing unless they say otherwise; at a café the barista did, so a bad cup speaks about the café and only faintly about the coffee or roaster. A roaster's style emerges from the descriptors people use about its coffees that the green cannot explain (baked, ashy, hollow, and their opposites), never from a declared field. Details in `docs/RECOMMENDATIONS.md`, Attribution.
- **Disliking a roaster or origin for non-taste reasons** (ethics, sourcing practices, politics) is not taste. The person says so directly; their coffees then sort last, grayed, labelled with the reason given. Nothing is hidden and it is reversible. Descriptors the person gives for such a cup still feed their taste model; the cup's overall rating does not. This is the one place a person states a rule rather than a verdict on a cup, because it is a rule they genuinely hold.
- **Drinks are part of taste, not a mode**: a person's cappuccino cups and their filter cups are different points and cluster apart on their own; nothing special-cases them. A coffee has no single aggregate and no single anchor, only per-drink-class ones, so a washed Kenya someone disliked in a cappuccino never touches its verdict as filter. Nobody is asked for a separate descriptor vocabulary per drink: flavour is flavour, milk adds a few descriptors to the one vocabulary, and free descriptors handle the rest until usage promotes them. Details in `docs/RECOMMENDATIONS.md`, Drinks.
- **Recommendations**:
  - Similar-taste review surfacing: rank a café's or coffee's reviews by similarity between reader's and author's taste. Cheap, no LLM.
  - Coffee/café suggestions: nearest neighbors to any of the person's anchors, filtered by availability (local roasters, cafés nearby, bags stocked locally). Explanations name the anchor or the taste ("because you loved X", "one of your tastes: clean washed Ethiopians").
  - Attributes beyond flavor (roaster size, certifications, bag design) surface as preferences through skew between what a person loved and what they tried, and are explained the same way. Popularity may be a personal preference; it is never a ranking.
  - A roaster's style as a personal preference is found the same way, with a second check: the person's dislikes fall on that roaster more than their tries would predict, and the descriptors in those dislikes are roast-caused and match how others describe that roaster. Explained with its evidence ("three of X's coffees you found baked and flat; others describe X's roasts that way too"). One cup says nothing about a roaster.
  - Experiments: items at a controlled distance from the person's anchors, always labeled as such. Three regions: near a loved anchor (recommend), near a disliked anchor (never offer, however novel), far from every anchor (genuinely new; offered only along dimensions the person is curious about). Feedback on experiments maps the user's boundaries.
  - AI summaries: never per user. Two forms are open (see Open questions): one general summary per entity that names disagreement between taste groups, or one summary per entity per taste mode. Either is cached and regenerated when enough new reviews arrive, so cost is constant per entity. A private per-user taste profile summary, shown only to its owner, is separate and cheap.
  - Collaborative filtering later, once user overlap is sufficient.
  - Approach in `docs/RECOMMENDATIONS.md`.

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
- Coffee page summaries: one general AI summary per entity that names disagreement between taste groups (plus reviews ordered by taste similarity), or one summary per entity per taste mode. Both open; decide once real reviews exist.
- Store listings via a native shell (Capacitor or Tauri): a possible future, decided on go-to-market grounds (discoverability, legitimacy with roasters), not a design constraint.
- Duplicate matching UX when logging a coffee without a printed ID: to be designed.
- Assessing home brews from their recipes (was this cup under- or over-extracted): deferred until enough brews are logged; for now the person's own assessment of each brew is the signal. Finickiness per brew method is derived from those assessments, never asked as a rating.
- Whether a rating given for non-taste reasons ("tasted great, 2/10 though") is excluded from the coffee's community aggregate, or counted with its reason shown.
- Moderation process and what "trusted contributor" concretely means.
- Whether public reviews are included in exports as aggregates only, or not at all.
- Roaster validation: talk to a few roasters before building B2B tools, including whether they would print a QR/ID on bags.
- Name for the identifier scheme.
- Drink classes for verdicts: whether black espresso is its own class from day one or joins filter until the data splits them.

## Builder context

Solo developer, building with AI assistance, no funding. Platform is decided: a mobile-first PWA; stores are a possible future. The stack and the standing rules are in `docs/ARCHITECTURE.md`. Constraints that follow from the manifesto and the situation:

- **Self-hostable by anyone.** Standard, portable components (e.g. plain Postgres, containerized deployment); no logic locked into proprietary vendor features. Whether the flagship instance runs on managed or self-operated infrastructure is a separate choice, made on cost and operational load, not principle.
- **Contributor-friendly.** One-command local setup; tools with enough adoption that new contributors and AI assistants are already fluent in them.
- **Stable and well-maintained**, but not necessarily conventional. Excellent tools with real momentum are preferred over defaults chosen for familiarity; tools with a single maintainer or an uncertain future are avoided.
- **Permanent public identifiers** (see Identifiers) mean URL structure and ID generation are decided once.
