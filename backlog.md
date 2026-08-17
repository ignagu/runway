# Backlog

Things that are known and thought through but not built. Newest first. Nothing
here is a commitment — it's a record so the reasoning doesn't have to be
reconstructed later.

---

## Open

### Itemised expenses, with a link per expense

**The problem.** A category holds a single number. A trip needing two flight
bookings can only record their sum, so the two can't be told apart. And a hotel
found on Booking.com but not yet booked has nowhere to live — there's no way to
save the link and come back to it.

**Model.** Replace the flat map with a list:

```js
// now
breakdown: { flights: 1200, lodging: 600 }

// proposed
expenses: [
  { id: "e1", cat: "flights", label: "Outbound",  amount: 700, url: "" },
  { id: "e2", cat: "flights", label: "Return",    amount: 500, url: "" },
  { id: "e3", cat: "lodging", label: "",          amount: 600, url: "https://…" }
]
```

Category totals become derived — the sum of that category's expenses. Trip
totals, the drill-down proportion bars, the dashboard figures and the
month-by-month chart all read derived values, so their own logic is unchanged.

Line items **replace** the per-category number rather than sitting alongside it.
Keeping both would mean two sources of truth for one figure and a reconciliation
on every edit.

**Links belong to an expense**, not to the trip — a link is for *this* hotel or
*that* flight. No trip-level link list.

**Security — do not skip.** The app renders through `innerHTML`, so a stored
`javascript:` URL would be live code, not inert text. Accept `http` and `https`
only and drop anything else at the point of saving *and* rendering. Render as
`target="_blank" rel="noopener noreferrer"`, and show the hostname
(`booking.com`) rather than the raw URL, which keeps the card readable and makes
a disguised link obvious. The app itself still makes no network requests — a
stored link does nothing until it is tapped.

**Migration.** Each non-zero `breakdown[k]` becomes one expense in that category
with an empty label. Older backups must still import, and the new shape has to
survive a Backup → Restore round trip unchanged. `migrate()` already tolerates
partial shapes and absorbs unknown category keys; this extends it.

**Touch points** (all in `index.html`): `emptyBreakdown()`, `cats()`,
`catByKey()`, `tripTotal()`, `migrate()`, `tripForm()` / `readForm()`,
`tripCard()`.

**UI sketch.** Expense rows in the trip form — category, label, amount, optional
link — with `+ Add expense` and a remove control per row. The drill-down groups
by category and lists each line under its category bar. Custom categories keep
working as they do today.

**Worth deciding when it's built:** whether an expense carries its own date
(useful for a deposit paid months before the trip), and whether a "booked" flag
per expense would be more useful than the trip-level status.

---

## Recently fixed

### Start and End date pickers overlapped

**Symptom.** On iPad the two date fields collided; they looked fine on desktop.

**Cause.** The row was `grid-template-columns: 1fr 1fr`. Grid items default to
`min-width: auto`, so a track cannot shrink below its content's intrinsic
minimum width. A native `input[type=date]` at the app's mandatory 16px font has
a wide intrinsic minimum — wider in iOS Safari than in Chromium, which is why
desktop testing missed it. Each field demanded more than half the row.

**Fix.** `repeat(auto-fit, minmax(190px, 1fr))` plus `min-width: 0` on the grid
items and inputs. This is content-driven rather than a guessed breakpoint: the
row keeps two columns where two real controls fit (iPad, desktop) and drops to
one where they don't (phones).

The same pass fixed a separate overflow: the category grid pushed the page
~5px wide at 375px and ~60px at 320px, because a fixed-width amount input and a
non-truncating label couldn't shrink.

**Note for anyone re-testing:** Chromium cannot reproduce the original bug — its
native date control is narrower. The layout suite emulates a wider control by
forcing `min-width: 190px`, which is a proxy, not proof. The real check is an
iPad.

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
