# Backlog

Things that are known and thought through but not built. Newest first. Nothing
here is a commitment — it's a record so the reasoning doesn't have to be
reconstructed later.

---

## Open

### Group trips by year

The Trips tab is one flat list sorted by total, so a 2026 trip sits next to a
2028 one with nothing to separate them. Group under year headings — year, trip
count, year total — keeping the sort by total inside each group. Trips route to
a year by start date today (`tripYear()`, `index.html:939`); undated ones need
a home, either the planning year as now or their own "No dates yet" group.

Pairs with the year-context item below: if the Trips tab knows which year the
dashboard is on, that group is the one to open on.

### Dates: end after start, same-day default, and trips with no fixed dates

Three related things, smallest first.

- **End cannot precede start.** Today the two are silently swapped on save.
  Set `min` on the end field from the start value so the picker won't offer an
  earlier day.
- **Picking a start defaults end to the same day.** A one-day trip is then no
  extra taps, and a longer one is an edit rather than an entry.
- **A trip may have a length but no dates.** "Two weeks somewhere in spring
  2028" is a real plan worth budgeting, and forcing exact dates to record it
  invents precision that doesn't exist.

The third is the substantial one. A trip would carry either exact dates *or* a
duration plus a rough window (a season, month or quarter of a given year).
Open questions to settle before building:

- Which month does an undated trip land in on the month-by-month chart —
  spread across the window, or the window's first month?
- How does the calendar show it? It has no days to mark. A band across the
  season, or simply absent from the grid and present only in the trip list.
- Does it still route to a year the same way? The window's year is the obvious
  answer.

### Public holidays for Spain and Sweden, five years out

Only 2026 is baked in. Extend through 2031.

**Sweden is fully rule-based** and should be computed, not typed: fixed dates
(1 Jan, 6 Jan, 1 May, 6 Jun, 24/25/26/31 Dec), Easter-relative days
(Långfredagen −2, Påskdagen, Annandag påsk +1, Kristi himmelsfärd +39,
Pingstdagen +49), Midsommarafton as the Friday falling 19–25 June with
Midsommardagen the day after, and Alla helgons dag as the Saturday falling
31 Oct – 6 Nov. Sweden does not move a holiday that lands on a weekend.

**Spain is mostly rule-based but not entirely.** National fixed days and the
Easter-relative Jueves Santo (−3) and Viernes Santo (−2) compute cleanly, as do
the Comunidad de Madrid and Madrid local days (2 May, 15 May, 9 Nov). What does
*not* compute is which national days get transferred when they fall on a
Sunday — that is set annually in the BOE *calendario laboral*, and the same
mechanism produced the two "observed" entries already in the 2026 list. The
regional and local selections are likewise confirmed year by year.

So: implement the Computus (anonymous Gregorian algorithm) and derive
everything derivable; mark any Spanish transferred day as provisional until the
official calendar for that year exists, rather than asserting a date the app
cannot know. Do not hand-write dates from memory — that was the standing
instruction from the original spec and it still holds.

### Carry the dashboard's year into a new trip

The dashboard has a year switcher; the Trips tab ignores it. Someone looking at
2028 and then adding a trip almost certainly means 2028, but gets whatever the
date picker defaults to.

Make the selected year visible on the Trips tab and use it as the default for a
new trip — prefilling its dates or its rough window — while leaving it trivially
changeable. It is a default, not a constraint: adding a 2029 trip while looking
at 2028 must stay a one-step action, not a fight.

---

## Shipped

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

---

## Deferred — decided, not scheduled

### Google Drive sync

Reviewed and achievable entirely client-side: Google Identity Services for the
token, Drive REST v3 with the `drive.file` scope (which only touches files the
app itself creates, is classed non-sensitive, and so needs no Google
verification review). No backend. Local storage would stay the source of truth
with Drive as a mirror.

Held deliberately. The three blockers, recorded so this isn't re-argued:

1. Needs a Google Cloud OAuth client ID, which only the repo owner can create.
2. OAuth popups are unreliable in an iOS home-screen PWA — they can bounce to
   Safari and fail to return. Redirect flow works but navigates the app away and
   back.
3. Two devices editing offline is last-write-wins unless Drive's revision id is
   tracked and divergence is surfaced.

Until then, Backup/Restore is the sync path: the share sheet into Files/iCloud
on iOS, a normal download elsewhere.
