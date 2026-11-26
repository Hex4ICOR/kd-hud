# KnuckleDown heads-up display

A single read-only page for the iPad on session days. It reads that day's
sign-up sheet (built by `build_signup_sheet_v2.py`) and shows NOW / NEXT / THEN
full screen, plus a session timer.

- Open: `https://hex4icor.github.io/kd-hud/?sheet=<spreadsheet-id>`
- Try it with sample data: `https://hex4icor.github.io/kd-hud/?demo=1`
- Optional: `&mins=20` changes the session length (default 15).

## What it does and doesn't do

- Reads cells A1:C60 of the "Sign-up" tab every 10 s through Google's public
  gviz endpoint. The sheet must have link sharing on (it already does on the day).
- Only ever shows initials and slot times. It never reads the Focus column and
  never writes to the sheet: Now / Done are still set in the sheet.
- The next free slot says "Click here to join the queue". That and "Book a time
  for later" open the sheet at that slot; initials are typed in the sheet.
- Shows "Live · updated N s ago"; after 35 s without a reply it greys out and
  says when it last updated. A failed read is never shown as an empty queue.
- Timer start time is kept in this browser (localStorage), so it survives
  switching to the intake app and reloads. No health or form data passes
  through this page, and none may be added to it.

## Wellbeing form link

Add `&form=<tailnet host:port>&site=<site slug>` to the bookmark. Tap a person to
open the form from the Mac in a frame over this page. The frame is a different
origin, so the browser stops this page reading anything in it, and the form sends
only three words: ready, saved, next. "Screened" is held in this browser as slot and
initials, for that day, and is never sent to Google, GitHub or anywhere else.
Add `&embed=0` to open the form in a new tab instead and mark people screened by hand.

This repo holds no client data. The sheet id lives only in the bookmark on
Kenn's iPad.

## Sessions app (app.html and v.html)

The fixed-address replacement for the daily sheet link. The old display above still works
and stays as the fallback.

- **iPad app:** `https://hex4icor.github.io/kd-hud/app.html`. Add it to the Home Screen once.
  Sign in once with the KnuckleDown database login; it stays signed in.
  Sample data, no sign-in: `app.html?demo=1`.
- **Venue page:** `https://hex4icor.github.io/kd-hud/v.html?v=<venue>` (for example `?v=tca`).
  This is the link a venue forwards. It shows what the session is, which times are free or taken,
  and whether the chair is busy and for about how long. People add initials to a free time.
  Sample: `v.html?demo=1`.

How it stays live:

- Venues, session days and slots live in the KnuckleDown Supabase project (tables `sess_venues`,
  `sess_days`, `sess_slots`). "Today" is always worked out in Glasgow time on each refresh, so the
  board and the venue page never show an earlier day. The board refreshes every 5 seconds, the
  venue page every 10.
- A day is a venue, a date and a from/until time. Free slots are worked out from those and the
  venue's slot length; only taken slots are stored. Change any of it in Set-up on the iPad.
- With no session set for today the board offers "Start today's session" (venue, from, until).
- Changes made on the iPad show at once and are queued if the connection drops, then sent when
  it returns. The footer says Live, Saving, or when it last updated.

What is and is not stored:

- Initials, slot time, status (waiting, in the chair, done, no-show), and two yes/no ticks:
  screened and paid. Nothing else. No names, no contact details, no health or form content.
- The tables are closed to everyone except the signed-in owner (row level security through
  `is_app_owner()`). The venue page can only call two functions: `sess_public` (times taken and
  chair status, never initials) and `sess_public_join` (add initials to a free time on an open day).
- The wellbeing form is still the Mac form in a frame (the venue's "Wellbeing form address"),
  with the same three words as above. "Paid" is set by Kenn's press-and-hold after seeing the
  customer's confirmation; the QR code is the venue's Stripe payment link.
- `sessions-config.js` holds the project address and the publishable key. That key is meant to be
  public; it gives no access to the tables.

`vendor/` holds pinned copies of supabase-js 2.45.4 and qrcode-generator 1.4.4 so the app does
not depend on a CDN on the day.
