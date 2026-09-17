# Recommendations

How recommendations are meant to work. Concept level; the mechanics get refined in a dedicated session. `docs/CONCEPT.md` (Taste model) holds the product framing; this document holds the approach. Draft, 2026-09-17.

## Representation

Every entity (coffee, café, roaster) carries structured attributes (canonical descriptors, process, origin, variety, roast, amenities, roaster size, certifications, and similar facts) plus embeddings of its text (descriptions, free descriptors, reviews). Together these place the entity as a point in a similarity space. Structured attributes are the explainable half; text embeddings capture what nobody named.

## A person is a set of points, never an average

Taste is plural. Someone may love heavily fermented coffees from anywhere, clean washed Ethiopians, and every natural Kenyan; averaging those into one vector describes nobody. The same is true of cafés: a counter-culture community spot and a posh high-end one can both be genuine tastes of one person.

- Loved entities are kept as individual anchors, and so are disliked ones. Recommendation is attraction to loved anchors minus repulsion from disliked ones. "I hated it" is as valuable as "I loved it".
- A clustering method that discovers the number of clusters itself (density-based such as HDBSCAN, or hierarchical with a distance cutoff; never a fixed-k method) groups anchors into named tastes for explanation. One taste or twenty, both are fine. Naming a taste needs a few points; until then the raw anchors still drive recommendations.
- Nobody is asked to state rules about their own taste. People often do not know why they like something, and the theories they hold about themselves ("I hate Arabica") are often wrong. The system stores verdicts on specific cups, and patterns are found across cups later.

## Reading notes

A free-text note is read into descriptor observations, each with an intensity and a verdict: "fermented, high, too much for this person". The person confirms what was read about this cup, never any rule about themselves; a skipped confirmation is kept at lower weight; a rating with no descriptors is still a valid point because the coffee's own facts are attached. Observations attach to the specific coffee, so conditional preferences (fermented Ethiopians yes, fermented Brazils no; acidity yes, Pink Bourbon's acidity no) emerge from where the points sit. From a single point the system can only say "you did not enjoy this one"; after a few it can say more, and it should not claim more than it has.

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

The similarity function between structured and embedded parts; how anchors are weighted by rating and recency; cluster thresholds; how declared curiosity and observed curiosity combine; the onboarding quiz's role before the diary has data; the summary form.
