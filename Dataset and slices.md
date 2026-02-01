# How the VK-LSVD dataset is structured

**VK-LSVD** is a log of short video impressions. Each line in 'interactions' corresponds to **one impression of one video to one user**, including the user's reaction to that impression.

---

## Core Entities

- **user_id** — user (anonymized)
- **item_id** — video (anonymized)

---

## Interactions

For each impression, the following are recorded:

- **Implicit Signals:**
`timespent` (`uint8`) — how many seconds the user watched the video (the main implicit signal)

- **Explicit Actions:**
`like`, `dislike`, `share`, `bookmark`, `click_on_author`, `open_comments` (boolean values)

- **Impression Context:**
`place`, `platform`, `agent` (usually ignored in basic models)

---

## Important Features

- Each `(user_id, item_id)` pair occurs **exactly once**
- Data is **not Session-based** — a single session can be represented by multiple rows

---

## Temporal Structure

- There may not be an explicit `timestamp`
- Time is specified by the order of rows and files:
`week_00 → week_01 → … → week_24 (train) → week_25 (validation)`
- Events from different users are mixed, the log is in global time

---

## Train / Validation

- **Train** — past weeks, used to train the model
- **Validation** — next week (future), used for fair evaluation

---

## Metadata

- **users_metadata:** age, gender, geo, activity_rank
- **items_metadata:** author_id, duration, popularity_rank
- **item_embeddings:** content-based Video embeddings (64 components, can use the first n)

---

## Subsamples

Examples:

- `up0.001` — 0.1% of users
- `ip0.001` — 0.1% of popular items by `train_interactions_rank`

The time split is preserved: 25 weeks of training + 1 week of validation

---

## Slicing

**Notations ur / up / ir / ip:**

- **Users:**
- `urX` — random users, X = share, e.g. `ur0.01` → 1% of users
- `upX` — popular users, X = share of the most active, filtered by `train_interactions_rank`

- **Items:**
- `irX` — random items
- `ipX` — popular Items, filtered by `train_interactions_rank`

---

## train_interactions_rank

Key variable — rank by number of interactions in the train:

- `rank = 1` → most popular/active
- The higher the number → the less popular the item

Usage:

- For users — determines activity
- For items — determines popularity

Advantages of a precomputed field:

- No need to recalculate popularity each time
- Allows for fair cross-sections of activity and popularity data
