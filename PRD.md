# Japan Haul — Gashapon PRD

## Overview

A personal desktop web app that gamifies unpacking a Japan shopping haul. Instead of opening everything at once and feeling overwhelmed, the user gets one surprise per day — dispensed by a gashapon (capsule toy) machine. The app turns a drawer full of purchases into a slow, savored ritual.

**Format:** Single HTML file, desktop browser, no server, no dependencies.  
**Audience:** One user (Shreya). No auth, no accounts.

---

## The Problem

Returning from Japan with a large haul creates a paradox: opening everything at once dilutes the joy of each individual item. The goal is to stretch the dopamine — one item per day, chosen randomly, with just enough ceremony to make it feel special.

---

## Core Mechanic

One spin per calendar day. Each spin dispenses one item from one category. After marking the item as opened, the machine locks until midnight. The user gets one "shake" (respin) per day if the result doesn't feel right — the capsule returns to the machine and a new one drops, with a possible category change.

---

## Categories

| Category   | Color         | Hex (suggested) |
|------------|---------------|-----------------|
| Kitchen    | Terracotta    | `#C4714A`       |
| Stationery | Indigo        | `#4A5B8C`       |
| Skin care  | Dusty rose    | `#C47E8A`       |
| Food       | Sage green    | `#6A8C5B`       |

Categories are fixed. Items within each category are pre-loaded from the user's list. No in-app inventory management UI.

---

## State Machine

The app has four states per day. State is persisted in `localStorage` keyed to today's date (YYYY-MM-DD). On date change, state resets to `READY`.

### READY
- Machine is active and waiting
- Coin slot is lit and clickable
- No capsule visible
- Shake counter: 1 remaining

### DISPENSED
- Capsule has dropped and is sitting in the tray
- Capsule is colored by its category
- Capsule is not yet opened
- Category color is visible, but item name is not (still a surprise)
- Spin is "used" — pressing coin again does nothing

### REVEALED
- User clicked the capsule; crack animation plays
- Item name is shown inside the opened capsule
- Two action buttons visible:
  - **Mark as opened** → transitions to LOCKED
  - **Respin** → transitions to DISPENSED (new draw); only enabled if shake counter > 0

### LOCKED
- Machine is asleep for the day
- Coin slot is dark/inactive
- Shows the item that was opened today
- Countdown to midnight displayed ("back in Xh Xm")
- Resumes READY state next calendar day

---

## Spin Logic

### Category selection
- Only categories with at least one unopened item are eligible
- Selection is weighted by item count (more items = more likely to be drawn)
- On a respin, category may change — full re-draw from all eligible categories

### Item selection
- Uniform random from all unopened items in the selected category
- On a respin, previous capsule is returned (item is not marked used)

### Exhaustion
- When a category has no remaining items, it is silently excluded from draws
- When all items across all categories are opened: machine shows a permanent empty state — no more spins possible

---

## Functional Requirements

### FR-1: Daily spin gate
- App checks `localStorage` for today's date on load
- If a spin has been used and marked as opened today, show LOCKED state
- If a spin was used but not yet resolved (e.g., user closed browser mid-flow), restore the DISPENSED or REVEALED state from `localStorage`

### FR-2: Respin (shake)
- Each day the user gets exactly one respin
- Respin is available from the REVEALED state only (after capsule is cracked open)
- Using respin: previous item is returned to pool, new category + item drawn
- If respin already used today, the **Respin** button is shown but disabled
- Respin counter persists in `localStorage` alongside the daily state

### FR-3: Mark as opened
- Removes item from pool permanently (persists in `localStorage` as an opened-IDs set)
- Transitions to LOCKED state
- Machine shows what was opened today + countdown

### FR-4: Midnight reset
- On page load, compare stored date against today
- If different: reset spin state (READY), restore respin counter to 1
- Do not reset the opened-items list — those are permanent

### FR-5: Item pool
- Items are hard-coded in the HTML as a JS array
- Structure: `{ id, name, category }`
- Opened items tracked by ID in `localStorage`
- If all items opened: show empty-drawer state permanently

### FR-6: Session persistence
- All mutable state lives in `localStorage`:
  - `haul_date` — YYYY-MM-DD of last interaction
  - `haul_state` — one of: `ready | dispensed | revealed | locked`
  - `haul_capsule` — `{ itemId, category }` of current capsule (if in dispensed/revealed/locked)
  - `haul_shake_remaining` — `0` or `1`
  - `haul_opened_ids` — array of item IDs marked as opened

---

## UI / UX Spec

### Layout
- Full viewport, centered content
- Machine is the dominant visual element
- Max content width: ~480px, centered horizontally and vertically
- Background: off-white (`#FAF8F5`)

### Machine anatomy (top to bottom)
1. **Header** — small wordmark "日本ガチャ" or just "Japan Haul", muted, above machine
2. **Machine body** — rounded rectangle, slightly 3D-feeling with subtle shadow
3. **Drum window** — circular viewport into the ball reservoir (shows colored capsules tumbling)
4. **Coin slot** — rectangular slot below drum; glows when READY, dark when LOCKED
5. **Dispensing flap** — at the bottom of the machine body; capsule lands here
6. **Capsule tray** — below the flap; capsule sits here after dispensing, before being opened

### Capsule
- Rounded pill shape (classic gashapon style)
- Two halves: top dome + bottom dome, colored by category
- Closed state: solid color, no text
- Open state: two halves pulled apart, item name floats in the center
- Crack animation: brief shake → halves split apart → item name fades in

### Action buttons (REVEALED state only)
- Appear below the opened capsule
- **Mark as opened** — primary style, full width
- **Respin** — ghost/secondary style, below; shows "1 respin remaining" or is grayed out with "respin used"

### LOCKED state
- Machine body dims slightly
- Coin slot goes dark
- Capsule remains visible (opened, showing today's item)
- Below machine: countdown clock ("Next spin in 4h 32m")

### Empty state (all items opened)
- Machine body is dark/powered-off
- Text: "Drawer empty. Hope it was worth the trip."

---

## Animations

| Trigger | Animation |
|---|---|
| Click coin slot | Coin slides into slot (CSS translate + fade) |
| Drum spinning | Ball reservoir rotates 1-2 full turns, slows to stop |
| Capsule dispensing | Capsule drops from flap into tray (CSS translate + bounce easing) |
| Click capsule | Machine wobble → capsule shakes → halves split apart |
| Respin | Capsule bounces back up into flap → drum spins again → new capsule drops |
| Mark as opened | Capsule fades/shrinks slightly; machine dims |

All animations: CSS transitions or keyframes only. No JS animation libraries.

---

## Visual Design

- **Aesthetic:** Clean, slightly toy-like without being childish. Inspired by actual gashapon machines — rounded forms, bold category colors, white space.
- **Typography:** System UI sans-serif. Machine label in a slightly playful weight.
- **Machine color:** White or light gray body with subtle drop shadow.
- **Capsule colors:** Per category table above. Capsules are the only real color in the UI.
- **No icons** beyond what CSS can produce. No external assets.

---

## Out of Scope

- Mobile / responsive layout
- Sounds
- Inventory management UI (items are pre-loaded in code)
- Multiple users
- Streaks, history, or stats
- Push notifications / reminders
- Any server, backend, or syncing

---

## Item List

To be provided by user. Will be hard-coded as a JS constant in the HTML:

```js
const ITEMS = [
  { id: 1, name: "...", category: "food" },
  { id: 2, name: "...", category: "kitchen" },
  // ...
];
```

Categories must be exactly one of: `food`, `kitchen`, `stationery`, `skincare`.

---

## Open Questions

None — all resolved.
