# Backlog

Things that are known and thought through but not built. Newest first. Nothing
here is a commitment — it's a record so the reasoning doesn't have to be
reconstructed later.

---

## Open

Nothing open.

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
