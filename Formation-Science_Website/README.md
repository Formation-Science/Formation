# Formation Science — "Gravity" concept (dark/purple, V2)

Same business, same offers, same contact info as `Test_Website/` — a completely
different visual direction. Static HTML/CSS/JS, no build step, one file per page.

```
Test_Website_V2/
├── index.html            # the landing page (all CSS + JS inline)
├── favicon.svg           # tab icon (violet F-mark on near-black)
├── assets/
│   └── logo-mark.svg     # standalone logo mark, currentColor
├── privacy/index.html    # /privacy/  — EXAMPLE draft, needs a lawyer
├── terms/index.html      # /terms/    — EXAMPLE draft, needs a lawyer
└── README.md
```

## The concept

Dark, purple, "black hole" — the visual metaphor is gravity: chaos (missed
calls, un-chased quotes, unanswered reviews) gets pulled into order once it
enters your automation system. The hero's centerpiece is a small animated
orbital diagram — five automation "nodes" (missed-call text-back, quote
follow-up, reviews, after-hours, daily report) connected by curved paths with
a traveling pulse, all converging on a glowing "FORMATION / CORE SYSTEM"
center. It's doing double duty as both the requested workflow/flowchart
motif and the black-hole/gravity motif in one image, instead of treating them
as two separate decorations.

**This variant is dark-only by design** — no light mode, no toggle. That was
a deliberate call, not an oversight: a "black hole" aesthetic that flips to a
white background on a light-mode OS would undercut the entire concept. If you
ever want a light/dark toggle added back (like the first site has), say so
and it's a straightforward addition — background/ink/accent are all CSS
variables already.

## Typography

- **Unbounded** (display) — bold, geometric, the "futuristic" headline face
- **Manrope** (body) — clean, highly readable at length
- **JetBrains Mono** (labels, eyebrows, node text, stats) — reinforces the
  "system/data" feel everywhere the copy reads like a readout rather than prose

Deliberately different from `Test_Website/`'s Archivo/Plex pairing, so the two
concepts don't feel like the same page in different colors.

## A real bug this build hit (and the general lesson)

The first draft reused the CSS class name `.field` for two unrelated things:
the fixed full-viewport starfield background wrapper, *and* every form input's
wrapper div. Because both rules had equal specificity, the later one in the
stylesheet didn't fully override the earlier one — properties merge per-rule,
not per-selector — so every real form field also inherited the background
wrapper's `position:fixed; inset:0; pointer-events:none`. Visually it threw
giant fixed-position boxes over the page; functionally it made the entire
contact form **unclickable**, which for a lead-gen page is the whole ballgame.
Fixed by renaming the background wrapper's class to `.ambient`. Caught by
diffing `document.body.children` order and a form-field position audit before
calling this done — worth remembering: a class name collision is a silent
bug, it won't throw a console error, it just quietly breaks interactivity.

## Content

Identical facts to the first site: same 8 automations, same 5 guarantees,
same Founders/first-5 scarcity, same self-qualification section instead of
pricing, same FAQ, same contact info (480-560-8041 / hello@formationscience.com
/ 4539 N 22nd St Ste 7030, Phoenix AZ 85016), same anonymized "two-decade
industrial supplier" credibility line. Only the words and the visual system
changed, not the offer.

## Wiring the contact form (2026-09-18: Google Forms passthrough, temporary)

The form submits straight to a **Google Form** from the visitor's browser
via a hidden iframe — no n8n, no VPS, no database. This is deliberately
temporary: it's the simplest thing that captures real leads with zero
hosting cost while the business doesn't yet justify a Formation Science
VPS (that VPS is what would run n8n; it doesn't get spun up until there
are at least 5 paying customers). A Supabase-direct version (RLS-locked
`anon` key, no server either) and an n8n-webhook version were both built
and are kept as documented upgrade paths — see "Other backends already
built" below.

**Real, load-bearing limitation of this technique**: a hidden iframe POST
is cross-origin, so the page's JS can never read Google's response. It can
fire the submission but cannot confirm Google actually accepted it — the
"Got it" message shown to the visitor is optimistic, not a real
confirmation. This is an inherent limit of the iframe/no-CORS approach in
general (not something specific to this implementation), which is exactly
why it's meant to be temporary rather than the long-term answer.

### 1. Create the Google Form

1. In your Formation Science Google Workspace account, go to
   [forms.google.com](https://forms.google.com) → **Blank form**.
2. Add one question per field below, **in this exact order is not required,
   but each question's answer type matters** (Google auto-generates the
   entry ID either way, this just keeps things predictable):
   - First name — Short answer
   - Last name — Short answer
   - Company — Short answer
   - Phone — Short answer (don't use the built-in phone number validation —
     the value sent is digits-only, e.g. `4805551234`)
   - Email — Short answer (again, skip Google's built-in email validation —
     the real HTML5 `type="email"` check already ran on the site itself)
   - How many field techs? — Short answer (the site sends the selected
     option's text, e.g. `1–2`)
   - Biggest bottleneck — Paragraph
   - Consent — Short answer (receives literally `"Yes"` or `"No"`)
   - Consent text — Paragraph (receives the exact consent sentence shown at
     submission time — this is the TCPA paper trail; store it, don't skip it)
3. Turn off **required** on every question in the form builder. This form
   is filled by JS, not a person looking at it — Google's own required-field
   validation has no way to show an error back to the visitor (see the
   cross-origin limitation above), so a "required" question that arrives
   empty due to a bug would just make that submission silently vanish
   instead of failing loudly. All the actual validation already happened
   in the real form on the site (HTML5 `required`, phone digit count, email
   format) before this ever fires.
4. Click the **Settings** gear → **Responses** → turn on **"Get email
   notifications for new responses."** This replaces needing Web3Forms or
   any other notify-by-email service — it's built into Google Forms and
   needs no API key.
5. Click **Responses** tab → the green Sheets icon → **Create a new
   spreadsheet**. This gives you a live, readable table of every lead,
   which is genuinely nicer to work from day-to-day than digging through
   emails.

### 2. Find each question's entry ID

1. With the form open for editing, click **Send** (top right) → the `<>`
   (embed HTML) icon → copy the `src="..."` URL out of the `<iframe>` tag
   it shows you. Take that URL, strip everything from `?` onward, and
   replace the trailing `/viewform` with `/formResponse` — that full string
   is your `GOOGLE_FORM_ACTION`.
2. Open that same form's **live URL** (not the editor) in a normal browser
   tab, open DevTools (F12) → **Elements**, and search (Ctrl+F in the
   Elements panel) for `entry.` — each question's underlying `<input>` or
   `<textarea>` has a `name="entry.XXXXXXXXXX"` attribute. Match each one to
   its question by the surrounding label text.
   - Faster alternative: fill out the real form once with obviously-fake
     placeholder answers, submit it, then look at the confirmation page's
     URL or use DevTools' **Network** tab (filter for `formResponse`) to see
     every `entry.XXXXXXXXXX=value` pair sent in that one request — much
     easier to match to fields than reading raw HTML.
3. In `index.html`, replace `REPLACE_WITH_YOUR_GOOGLE_FORM_FORMRESPONSE_URL`
   with the URL from step 1, and each `REPLACE_WITH_ENTRY_ID` in
   `GOOGLE_FORM_ENTRIES` with the matching `entry.XXXXXXXXXX` string (the
   whole `entry.XXXXXXXXXX`, not just the number) from step 2.

### If you ever need to change a field later

Adding/removing/renaming a question in the Google Form does **not**
automatically update `index.html` — the entry IDs are copy-pasted, static
strings. Any time you edit the Google Form's questions, re-check the
`GOOGLE_FORM_ENTRIES` map in `index.html` against the form's current
`entry.XXXXXXXXXX` IDs (deleting and re-adding a question generates a
*new* entry ID, even if the question text is identical).

### Other backends already built (not currently wired up)

- **Supabase + Row Level Security, no server**: writes straight to a
  `website_leads` table using a public `anon` key restricted, at the
  database level, to insert-only on that one table — see
  `formation-science-context.md`'s 2026-09-18 entry for the exact RLS SQL
  and setup steps. The stronger long-term option once you're ready to move
  off Google Forms, still with zero hosting cost.
- **n8n webhook**: `n8n-workflow-website-lead-capture.json` (in the vault's
  `History` folder) — writes to Supabase and relays to Web3Forms
  server-side. Needs a running n8n instance (i.e. the VPS), so it's the
  option for once there are 5+ paying customers.

Until `GOOGLE_FORM_ACTION` is filled in, submitting the form falls back to
opening the visitor's own email app with a prefilled draft to
sales@formationscience.com, same graceful-degradation behavior as before.

A required consent checkbox now sits above the submit button — "I agree to
receive calls and texts from Formation Science about my inquiry, by person
or automation..." — both the checked state and the exact wording shown are
stored with the lead (`consent_given`, `consent_text` columns) as a
timestamped record, since a TCPA consent dispute turns on what was actually
shown and agreed to, not just a yes/no flag.

## Run it locally

```bash
python3 -m http.server 8080
```

Then open <http://localhost:8080>.

## Verified

Checked at 344px (Galaxy Fold, folded), 430px, 768px, and 1280px: zero
horizontal overflow, all 11 sections lay out cleanly, form fields are
genuinely clickable (see the bug note above), FAQ accordion works, contrast
ratios checked against the actual palette values — body text ~9-18:1 against
the near-black background, button text ~4.7-5.8:1 against the accent
gradient, all comfortably at or above WCAG AA.
