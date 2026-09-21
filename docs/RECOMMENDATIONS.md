# Recommendations

How recommendations are meant to work. Concept level; the mechanics get refined in a dedicated session. `docs/CONCEPT.md` (Taste model) holds the product framing; this document holds the approach. Draft, 2026-09-17.

## Representation

Every entity (coffee, café, roaster) carries structured attributes (canonical descriptors, process, origin, variety, roast, amenities, roaster size, certifications, and similar facts) plus embeddings of its text (descriptions, free descriptors, reviews). Together these place the entity as a point in a similarity space. Structured attributes are the explainable half; text embeddings capture what nobody named.

Each canonical descriptor is tagged with its likely cause: green (fruit, florals, acidity type, process character), roast (baked, flat, hollow, ashy, roasty, underdeveloped), freshness (papery, woody, baggy, stale), brew (sour and thin, bitter and drying, astringent), preparation (milk sweetness, texture, sour against the milk, and whatever tonic or syrup adds), or ambiguous. The tag is what lets a cup's verdict be attributed to the right thing (Attribution, below). Free descriptors inherit a tag when mapped to a canonical one; until then they are ambiguous.

## A person is a set of points, never an average

Taste is plural. Someone may love heavily fermented coffees from anywhere, clean washed Ethiopians, and every natural Kenyan; averaging those into one vector describes nobody. The same is true of cafés: a counter-culture community spot and a posh high-end one can both be genuine tastes of one person.

- Loved entities are kept as individual anchors, and so are disliked ones. Recommendation is attraction to loved anchors minus repulsion from disliked ones. "I hated it" is as valuable as "I loved it".
- A clustering method that discovers the number of clusters itself (density-based such as HDBSCAN, or hierarchical with a distance cutoff; never a fixed-k method) groups anchors into named tastes for explanation. One taste or twenty, both are fine. Naming a taste needs a few points; until then the raw anchors still drive recommendations.
- Nobody is asked to state rules about their own taste. People often do not know why they like something, and the theories they hold about themselves ("I hate Arabica") are often wrong. The system stores verdicts on specific cups, and patterns are found across cups later.

## Reading notes

A free-text note is read into descriptor observations, each with an intensity and a verdict: "fermented, high, too much for this person". The person confirms what was read about this cup, never any rule about themselves; a skipped confirmation is kept at lower weight; a rating with no descriptors is still a valid point because the coffee's own facts are attached. A note that blames or credits the brew ("ground too fine", "finally clicked on the third try") is read as the person's brew assessment, confirmed like any other observation. Observations attach to the specific coffee, so conditional preferences (fermented Ethiopians yes, fermented Brazils no; acidity yes, Pink Bourbon's acidity no) emerge from where the points sit. From a single point the system can only say "you did not enjoy this one"; after a few it can say more, and it should not claim more than it has.

## Attribution: what a cup is evidence about

What a cup tastes like is, roughly, the green plus what the roaster did to it plus how fresh the bag was plus how it was brewed plus what it was prepared as (milk, tonic, nothing) plus who is tasting. Every logged cup is evidence about all of these, with different weights. The weights come from where the cup was drunk and what the person said about it, never from the score alone.

- **At home**, the person controlled the brew. A cup they marked as a failed brew (or described with brew-caused descriptors) is strong evidence about their brewing and near-zero evidence about the coffee. A cup they marked as brewed right is evidence about the coffee. An unmarked cup sits in between.
- **The best cup shows the coffee's potential, but the person makes that call.** If ten brews of a bag went badly and the eleventh was superb, the eleventh dominates the coffee verdict only when the ten were marked or read as brew failures. If the ten were unmarked and described as baked or flat, that is the roast, and the eleventh is the anomaly. Taking the best of many cups automatically would credit noise: every coffee eventually gets a lucky cup.
- **The failed brews are kept.** A coffee that took ten tries is a finicky coffee, and that is a fact about it worth showing to similar-taste home brewers ("you loved this once you got it right; it took a while"). Finickiness is per brew method (immersion forgives what pourover punishes), so it is derived from attempts-before-success across people and per method, from the recipe already attached to each brew. Nobody is asked to rate ease of brewing, and it is never a score on the coffee; it is shown as a note ("people found this tricky in pourover") once there is enough evidence. The failed recipes on the same bag are also exactly the data a later brew-quality model needs.
- **Freshness is checked before the brewer is credited.** With a roast date on the bag and a date on each brew, "the eleventh cup was best" often reads as "this coffee needs two weeks of rest", not as a brewing breakthrough. Stale-crop descriptors (papery, woody, baggy) carry the signal even without dates.
- **A drink that does not match the roaster's intended use** (a filter roast drunk as a cappuccino) makes the cup strong evidence about that pairing and weak evidence about the coffee, the same way a failed home brew is. The mismatch is not hard-coded as wrong: some people love a bright Kenya flat white, and the system learns who they are rather than assuming.
- **At a café**, the barista controlled the brew. A bad cup is strong evidence about the café, weak evidence about the coffee, and weaker still about the roaster. For a milk drink even less reaches the coffee, since milk type, texture, and ratio were the barista's too. Café evidence accumulates over visits too, since shifts and machines vary. Similar-taste review surfacing (below) already shows "people with your taste were disappointed here" without any further mechanism.

**A roaster's style is learned, never declared.** Roasting is multi-axis (a light roast can be slow and cool or fast and hot, giving very different cups), and roasters do not publish profiles, so no field could capture it. Instead: across all of a roaster's coffees, the part of the descriptor profile that the green facts (origin, process, variety, farm) fail to explain is that roaster's residual. Aggregated over many coffees it becomes a fingerprint ("tends toward ashy", "tends toward floral and green"). The cleanest instance is the same lot from the same harvest roasted by two roasters, which is common across the community even though it is rare in one person's diary; the residual does not require exact pairs. The fingerprint is computed community-wide and is also what positions a new, unreviewed release from that roaster before anyone has logged it.

**Whether a roaster's style suits a person** is then the existing skew test with a second check. First, do the person's dislikes fall on that roaster more than their tries would predict? Second, are the descriptors in those dislikes roast-caused, and do they match the roaster's fingerprint? Both together justify "X's roasting does not work for you", shown with its evidence count. If the disliked descriptors are green-caused (too fermented, too fruity), the origins are blamed, not the roaster. One cup never justifies a statement about a roaster; the explanation always says how many it rests on.

**Non-taste reasons to avoid a roaster or origin** (ethics, sourcing practices, politics) are declared by the person, not inferred. Their coffees sort last, grayed, labelled with the reason given; nothing is hidden and it is reversible. Descriptor observations from such cups still count, with their per-descriptor verdicts: "tasted great, 2/10 though" says the person enjoyed the coffee's characteristics. The cup's overall rating does not become an anchor for that person. Whether it also stays out of the coffee's community aggregate is open (`docs/CONCEPT.md`, Open questions).

## Drinks

A verdict is always about a coffee in a drink. Ratings and descriptor observations are keyed by coffee and drink class (black filter, black espresso, milk, other; the specific drink is recorded and used once it has enough data of its own). Nothing averages across drink classes: a coffee has no single aggregate and no single anchor, only per-class ones, shown relative to the reader's taste ("as filter, most people with your taste loved it; as a milk drink, three tries, mixed"). Never a score per class either; that would be a ranking with extra steps (manifesto 1).

What carries across classes is knowledge, not verdicts: the coffee's facts, the roaster's fingerprint and intended use, and the person's descriptor-level taste (likes bright, dislikes ashy). Those act as a prior for a class the person has little data in, at lower weight than same-class evidence, and suggestions built on them are experiments, labelled as such. Someone with only filter cups in their diary can still be offered beans for a cappuccino; the offer says why and how little it rests on.

The person never chooses a mode. The drink is a dimension of the point, so nearest-neighbour search separates classes on its own, and a recommendation can say "for your filter brews: X; for your milk drinks: Y" without being asked.

Descriptors are optional in every class. A rating with the coffee and drink recorded is a valid point on its own, so people who would never pick descriptors for an espresso tonic still get recommendations from what they rated.

## Recommending

- Nearest neighbours to any loved anchor, away from disliked ones, filtered by availability (local roasters, cafés nearby, bags stocked locally).
- Explanations name the anchor or the taste: "because you loved X", "one of your tastes: clean washed Ethiopians". Attributes beyond flavour (roaster size, certifications, bag design) surface as preferences through skew between what a person loved and what they tried, and are explained the same way. Popularity may be a personal preference dial; it is never a ranking (manifesto 1).
- Similar-taste review surfacing: a café's or coffee's reviews are ordered by similarity between the reader's and each author's taste.
- Three regions of the space: near a loved anchor (recommend); near a disliked anchor (never offer, however novel); far from every anchor (genuinely new; offered only as an experiment, and only along dimensions the person is curious about).
- Curiosity is separate from liking. The person can declare what they are curious about (new processes, varieties, roast styles, brew methods) and what they are not; which experiments they take up refines it. Experiments are always labelled as such (manifesto 1).
- Collaborative filtering ("people who liked what you liked also liked") waits until enough rating overlap exists; when it comes it is a scheduled job writing to a table.

## Where language models are used

Per-record, cached, regenerated on thresholds, never per request:

- Reading a note into descriptor observations (above).
- Reading a typed name or bag photo into search terms for candidate matching when logging a coffee without a printed ID (UX to be designed separately).
- Entity summaries from reviews. Form open: one general summary per entity that names disagreement between taste groups, or one per entity per taste mode. Decide once real reviews exist.
- A private per-user taste profile summary, shown only to its owner.
- Assisting descriptor curation: mapping frequent free descriptors to canonical ones for human approval.

A language model never selects recommendation candidates and never measures distance; those are computed. It may phrase an explanation from the structured reasons. Reranking a short, already-selected list against the cached profile summary is a permitted later experiment, bounded by list length, adopted only if evaluation shows it helps.

Why not let a model recommend directly: it cannot measure distance, so experiments could not be labelled honestly; its explanation would be a story rather than the cause; it cannot be audited for manifesto 2; it invents coffees; it cannot be improved against an evaluation set; cost scales with users times diary length.

## Evaluation

Feedback on experiments and on recommendations ("was this useful?") is the evaluation set. The recommender is a feature with a measurable quality, not an afterthought.

## To refine in a dedicated session

The similarity function between structured and embedded parts; how anchors are weighted by rating and recency; cluster thresholds; how declared curiosity and observed curiosity combine; the onboarding quiz's role before the diary has data; the summary form; the attribution weights per cup context and the minimum evidence before a roaster fingerprint or a per-person roaster verdict is shown; assessing brew quality from recipes once enough brews exist; drink classes and when a specific drink graduates to its own class.
