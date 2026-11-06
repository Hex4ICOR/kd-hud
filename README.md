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
- Shows "Live · updated N s ago"; after 35 s without a reply it greys out and
  says when it last updated. A failed read is never shown as an empty queue.
- Timer start time is kept in this browser (localStorage), so it survives
  switching to the intake app and reloads. No health or form data passes
  through this page, and none may be added to it.

This repo holds no client data. The sheet id lives only in the bookmark on
Kenn's iPad.
