# Backlog

Things that are known and thought through but not built. Newest first. Nothing
here is a commitment — it's a record so the reasoning doesn't have to be
reconstructed later.

---

## Open

Nothing open.

---

## Shipped

### Trips grouped by year, with the planning year carried through

The Trips tab was one flat list, so a 2026 trip sat beside a 2028 one with
nothing between them. Trips now group under year headings carrying the count
and the year's total, still sorted by total inside each group.

The year switcher is shared with the dashboard: whichever year is on screen is
the one the Add button names, and a trip created without dates records that year
(`planYear`) rather than falling back to a hard-coded 2026. Changing the year
while a rough-plan draft is open moves the draft with it.

### Dates: end after start, same-day default, and plans without dates

- The end picker's `min` is the start date, so an earlier day cannot be chosen.
  A stored end before the start is clamped up rather than silently swapped, as
  it was before.
- Picking a start proposes the same day as the end, so a day trip needs no
  second pick and a longer one is an edit.
- A trip can instead carry a **rough plan**: a length and a period —
  "14 days, Spring 2028" — with no invented dates. One-tap presets cover
  weekend / 5 days / a week / 10 days / 2 weeks; periods are seasons, quarters
  or single months.

Decisions taken while building it:

- **Seasons name their months** (Spring (Mar–May), Winter (Jan–Feb)). "Winter
  2028" is otherwise ambiguous across the year boundary; the label removes the
  guess rather than the app making one.
- **A rough plan lands in the first month of its period on the chart** —
  predictable, and stated in the form. It counts against its year normally.
- **It does not appear in the calendar at all.** There are no days to mark, and
  inventing some would be false precision. The trip list is where it lives.
- Switching a trip to exact dates clears the plan, so there is never both.

### Spanish and Swedish holidays through 2031

2026 stays the verified baked list. 2027–2031 are computed from rules, never
typed.

**Sweden is exact** — fixed days, Easter offsets via the computus,
Midsommarafton as the Friday falling 19–25 June, Alla helgons dag as the
Saturday falling 31 Oct – 6 Nov, and no weekend transfers.

**Spain is computed but partly provisional.** The fixed nationals, Jueves and
Viernes Santo, and the Madrid regional and local days all derive cleanly. What
cannot be derived is which national day moves when it falls on a Sunday — that
is set annually in the BOE *calendario laboral*. Those transfers are emitted on
the Monday, flagged provisional, labelled as such in the day detail, and
explained by a line in the calendar. A regional or local feast landing on a
Sunday keeps its real date and is simply not an extra day off.

Both rule sets were checked by generating 2026 and diffing against the verified
baked list — they reproduce it exactly, which is the evidence that the rules are
right. The computus was separately checked against eight known Easter dates.
Navigating past 2031 says so rather than showing a year that merely looks
holiday-free.

### Itemised expenses, with a link per expense

A category used to hold a single number, so two flight bookings on one trip
could only be recorded as their sum, and a hotel found on Booking.com had
nowhere to live until it was booked.

Trips now hold a list of expenses — category, label, amount, optional link —
and category totals are derived from them. Trip totals, the proportion bars,
the dashboard figures and the month chart all read derived values, so their
logic was untouched.

Decisions taken while building it:

- **Line items replace the per-category number** rather than sitting alongside
  it, so there is one source of truth for every figure.
- **A link belongs to an expense**, not to the trip.
- **A zero-amount expense with a label or a link is kept** — that is the
  "found the hotel, haven't booked it" case the feature exists for. Only rows
  that are blank in all three respects are dropped on save.
- An expense carries those four fields and nothing else; trip status remains
  the only booking signal. A per-expense booked flag and a per-expense date
  were both considered and left out — the date in particular would force the
  month chart to choose between bucketing by trip start or by expense date.
- The drill-down only lists categories that have expenses, and skips the
  sub-line for a lone unlabelled, unlinked expense, since the category row
  already says it.

**Links are restricted to http/https**, checked when saving and again when
rendering, and shown as their hostname. A `javascript:` URL smuggled directly
into a data file is stripped on load rather than rendered — the app builds its
DOM with `innerHTML`, so that would otherwise be live code.

### Start and End date pickers overlapped

**Symptom.** On iPad the two date fields collided; they looked fine on desktop.

**Cause.** The row was `grid-template-columns: 1fr 1fr`. Grid items default to
`min-width: auto`, so a track cannot shrink below its content's intrinsic
minimum width. A native `input[type=date]` at the app's mandatory 16px font has
a wide intrinsic minimum — wider in iOS Safari than in Chromium, which is why
desktop testing missed it. Each field demanded more than half the row.

**First fix, insufficient.** `repeat(auto-fit, minmax(190px, 1fr))` plus
`min-width: 0`, meant to keep two columns where two real controls fit and drop
to one where they don't. It still overlapped on a real iPad at roughly 213px
per column: iOS Safari does not let `min-width: 0` shrink a date control below
its intrinsic minimum, and that minimum is larger than the 190px the test
emulated.

**Actual fix.** Start and End always stack, one row each. The app is capped at
560px, so there is no width at which two native date controls reliably fit
side by side, and any threshold is a guess that had already been wrong twice.
Stacking removes the failure mode by construction.

The same pass fixed a separate overflow: the category grid pushed the page
~5px wide at 375px and ~60px at 320px, because a fixed-width amount input and a
non-truncating label couldn't shrink.

**Lesson worth keeping:** Chromium cannot reproduce this — its native date
control is narrower, and it honours `min-width: 0` where iOS does not. Emulating
a wider control by forcing `min-width` is a proxy that passed while the real
device still failed. Where a native control's size cannot be measured, prefer a
layout that does not depend on knowing it.
