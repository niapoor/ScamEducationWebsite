# Phishing examples

This folder holds the individual simulated phishing emails shown by the
viewer at `/resources/email-based` (`../index.html`). Each example is its
own plain HTML file, so examples can be added, edited, or removed one at a
time without touching the viewer.

## Adding a new example

1. Copy `_TEMPLATE.html` to a new file in this folder, e.g.
   `21-fake-gift-card.html`. Numbering the file is a convenience for
   humans browsing the folder — the actual display order comes from
   `manifest.json`, not the filename.
2. Write the email: subject, from/to/date, and body, following the
   pattern already in the template and in `18-fedex-shipping-confirmation.html`.
3. Wrap each phishing indicator in an annotation (see below).
4. Add an entry to `../manifest.json`:

   ```json
   {
     "id": "fake-gift-card",
     "file": "21-fake-gift-card.html",
     "title": "Fake Gift Card Request",
     "description": "One sentence describing the scam for the viewer's UI."
   }
   ```

   `id` should be a short, unique, URL-safe slug — it's used for
   deep-linking to a specific example (`/resources/email-based#fake-gift-card`).

That's the entire process. The viewer doesn't need any code changes to
pick up a new example — it reads the manifest at load time.

## Keeping dates relevant

The "Date" field isn't the static text in the file — it's rewritten by
the viewer page every time an example loads, so it always reads as
some number of days before *whenever someone's actually looking at
it*, rather than a fixed date that quietly turns stale (and eventually
starts looking like it's from the future) as real time passes. That's
done entirely in the parent page's JS, not inside the example itself,
since these examples load in a sandboxed iframe with no scripts.

Structure the Date field like this:

```html
<dd class="email-date" data-days-ago="4" data-time="9:34 AM">Thu, Jul 16, 2026, 9:34 AM</dd>
```

- `data-days-ago` is a small constant (1–6) — never 0. The viewer computes
  the actual calendar date as "today minus this many days" and replaces
  the element's text with it, in the same `Ddd, Mon DD, YYYY` format.
  Vary this a little across your examples (not all `3`, say) so a
  visitor browsing several of them doesn't see the exact same date on
  every one.
- `data-time` is preserved exactly as given — only the date changes,
  never the time of day.
- The plain text content of the `<dd>` (a normal-looking date, in the
  same format the real one will take) is just a fallback for anyone
  who opens the file directly rather than through the actual viewer
  page — it's never what a visitor to the real site actually sees.

If the same sent date needs to appear a second time in the body (a
"Date Received" line, an invoice-style "Date:" stamp, etc.), give it
the *same* `data-days-ago` as the header's `.email-date` so the two
never drift apart, using whichever of these matches how it's written:

```html
<span class="email-date-received" data-days-ago="4">16 July, 2026</span>
<!-- "DD Month, YYYY" — no weekday, no time -->

<span class="email-date-ddmmyyyy" data-days-ago="4">16-07-2026</span>
<!-- numeric "DD-MM-YYYY", as seen on some invoice-style templates -->

<span class="email-date-event" data-days-ago="4" data-days-ahead="3">Sunday, April 4th, 2026</span>
<!-- a FUTURE date (e.g. an invitation's party date), offset forward
     from the sent date by data-days-ahead (default 3) rather than
     from today directly — "Weekday, Month Dayth, Year" -->

<span class="email-date-plain" data-days-ago="95">April 24, 2026</span>
<!-- a plain "Month D, YYYY" mention elsewhere in the body (e.g. a
     scam's claimed date range) — independent of the header, just
     always recent and never in the future -->
```

## The annotation convention

Wrap the exact text or element that gives the scam away in a `mark`,
then follow it immediately with the explanation:

```html
<mark class="annotation" data-color="1">paypa1-secure-center.com</mark><span class="annotation-note" data-color="1">This isn't a PayPal domain — legitimate PayPal emails come from an address ending in @paypal.com.</span>
```

That's the whole convention:

- `mark.annotation` wraps the suspicious text itself, and nothing else.
- `span.annotation-note`, holding the plain-language explanation, comes
  **right after the closing `</mark>`, as its sibling — not nested
  inside it.** This matters more than it looks like it should: `mark`
  is an inline element, and `annotation-note` renders as a block (so
  the explanation appears as its own box below the flagged text).
  Nesting a block-level element inside an inline one is technically
  invalid HTML, and browsers paper over it by "blockifying" the inline
  element — which behaves inconsistently around line-wrapping, adjacent
  whitespace, and trailing content. Keeping `annotation-note` as a
  sibling instead sidesteps all of that: `mark` stays purely inline, and
  a paragraph mixing inline text with a block child is completely
  ordinary CSS with no edge cases to worry about.
- `data-color="1"` (matching on both the mark and its note, since
  they're separate siblings) cycles **1, 2, 3, 1, 2, 3...** across the
  annotations in a single email, in the order they appear — this picks
  yellow, pink, or blue so a note's color matches its flagged text at a
  glance, which matters more once there are several annotations visible
  on screen at once. Restart the count at 1 for every new file; it's
  independent of the numbered badge (which just counts up and never
  resets within a file).
- Numbering (the little circled digit), highlighting, and show/hide
  behavior are all handled by `example.css` automatically from there —
  never add a number, an id, or any JavaScript yourself.

Because numbering is generated with a CSS counter, you can add, remove,
or reorder annotations freely and they'll always renumber correctly.

A few practical notes:

- Annotate the smallest span of text that makes the point — a domain
  name, not the whole sentence around it — *unless* the red flag is the
  sentence as a whole (e.g. an urgency/threat line), in which case
  wrapping the full sentence is clearer.
- To annotate a button or link, wrap the `<a>` itself in the `mark`
  (the note that follows still explains the button, it just doesn't
  need to be nested inside anything to do that). Every link/button:
  - has **no `href` at all** — not even `href="#"`. An `<a>` with no
    `href` isn't clickable or navigable; nothing happens on click, tap,
    or keyboard activation. This is stricter than it needs to be for
    safety alone (see "Security model" below) — it's mainly so the
    example behaves the way the annotation describes: a link that
    truly does nothing when clicked, not one that jumps around within
    the page.
  - needs `tabindex="0"` so it's still reachable by keyboard (an `<a>`
    without `href` isn't focusable by default), and `role="button"`
    (styled buttons) or `role="link"` (plain text links) so screen
    readers still announce it correctly despite the missing `href`.
  - needs a `data-fake-url` attribute — hovering, focusing, or
    press-and-holding it shows that as a fake destination, the same way
    a real email lets you check where a link actually goes before
    clicking:

  ```html
  <mark class="annotation" data-color="2"><a tabindex="0" role="button" class="email-btn" data-fake-url="https://bash.video">Verify My Account</a></mark><span class="annotation-note" data-color="2">Explanation here.</span>
  ```

  Leave off `class="email-btn"` (and use `role="link"` instead of
  `role="button"`) for a plain inline text link instead of a styled
  button — the base link style (blue, underlined) applies
  automatically. **Don't make every example a "click the big button"
  scam.** Real phishing just as often asks you to reply, call a number,
  wire money, or hand over gift card codes with no link at all — vary
  it so the lesson isn't "buttons are the tell," it's "read carefully."
  A rough mix across all the examples: some with a styled button, some
  with only a plain text link, some with neither.
- Every link/button's note needs to explain *how* to check the real
  destination, not just that you should: on a computer, hovering the
  mouse over a link shows it (often in the browser's status bar); on a
  phone or tablet, press and hold the link to preview it the same way,
  without opening it. Work this into the explanation itself rather than
  appending it verbatim every time — see the existing examples for how
  it reads folded into different notes.
- Keep explanations short — one or two sentences for most annotations.
  Link/button notes are the deliberate exception, since they carry the
  extra hover/press-and-hold sentence above — three sentences there is
  normal, not a sign to trim.
- If the flagged text is immediately followed by trailing punctuation
  that's *not* part of the annotation — like the closing `>` after a
  sender's email address — wrap it in `<span class="annotation-suffix">`
  and keep it **inside** the `mark`, right after the flagged text:

  ```html
  <mark class="annotation" data-color="3">suspicious-domain.com<span class="annotation-suffix">&gt;</span></mark><span class="annotation-note" data-color="3">...</span>
  ```

  `annotation-suffix` is `display: inline-block`, which is what actually
  keeps it from picking up the mark's highlight and dotted underline —
  it isn't part of what's flagged, but it does need to stay inside the
  `mark` so it doesn't get separated from the flagged text if the line
  wraps. It currently assumes a meta-header (light gray) background —
  if you ever need it somewhere with a different background, add a
  matching override in `example.css`.
- If the flagged element already has its own strong visual treatment
  (say, text styled to look like a logo, sitting on a colored banner),
  the usual gold highlight/underline can compete with it rather than
  clearly marking it. Usually you still want *some* highlight visible
  around the badge, just not directly behind the styled text — do that
  by giving the text its own opaque background matching whatever it
  sits on, the same masking idea as `annotation-suffix` above, rather
  than removing the mark's highlight altogether:

  ```html
  <mark class="annotation" data-color="3"><span>Lowe's</span></mark><span class="annotation-note" data-color="3">...</span>
  ```
  ```css
  .survey-logo-box span { display: inline-block; background: #5b7aa8; text-decoration: none; }
  ```

  The mark keeps its normal background/underline, which now only shows
  in the small margin around the badge; the span's own opaque
  background (matching the banner behind it) masks the highlight
  specifically behind the logo text. `display: inline-block` is what
  excludes it from the underline too (a plain span with
  `text-decoration: none` alone isn't enough — see `annotation-suffix`
  above for why).

  For the rarer case where you want *no* highlight at all, not even
  around the badge, add `no-highlight` alongside `annotation` instead:
  `<mark class="annotation no-highlight" data-color="3">`. This
  removes the background, color override, and underline from the whole
  mark (the badge itself is a separate rule, so it's unaffected either
  way) — reach for this only when even a sliver of highlight would look
  wrong, since it's a blunter tool than the masking approach above.
- If a paragraph has plain text that needs to come *after* the
  annotation (not just before it), fold that text into the `mark`
  itself if it's short, the way the personal-information example does
  ("...using the link above." is part of the flagged sentence, not
  left dangling after it). `annotation-note` is block-level, so
  anything genuinely left after it in the same paragraph will render
  on its own line below the note box rather than staying attached to
  the sentence above it.

## Security model

These are simulated phishing emails on a public education site, so this
folder is written under the assumption that a mistake here should never
be able to do anything harmful. Several layers enforce that independently:

1. **The viewer loads every example in a sandboxed `<iframe>`**
   (`sandbox="allow-same-origin"`, deliberately *without* `allow-scripts`,
   `allow-forms`, `allow-popups`, or `allow-top-navigation`). This is the
   primary control — it's enforced by the browser and doesn't depend on
   anything in this folder being written correctly.
2. **Every link/button has no `href` at all.** An `<a>` with no `href`
   isn't clickable or navigable — nothing happens on click, tap, or
   keyboard activation, regardless of what a link's fake preview text
   says. (This used to be `href="#"` plus `pointer-events: none`; `#`
   was already harmless — it only ever jumped within the sandboxed
   document — but it wasn't literally *inert*, and `pointer-events: none`
   also blocked `:hover`, which is what the fake-destination preview
   needs. Dropping `href` entirely gets both: truly does-nothing on
   click, and still hoverable/focusable/pressable for the preview.)
3. **Response headers** (see the root `vercel.json`) send a strict
   `Content-Security-Policy` for everything under this folder, blocking
   scripts and form submissions at the network level as a third layer.

Given that, when writing an example:

- **Never add a `<script>` tag.** It will not execute, and its presence
  would be misleading to a future reader of the source.
- **Never give a link or button a real `href` — or any `href` at all.**
  See the annotation convention above for the full pattern
  (`tabindex`, `role`, `data-fake-url`). Don't rely on the sandbox
  alone, write it correctly.
- **Don't add forms.** If an example needs to show a fake login form,
  build it with plain `<input>`/`<label>` markup and no `<form>` tag or
  `action`/`method` attributes.
- **Don't reference external images or fonts.** Keep every example
  self-contained (this also matches the rest of the site, which never
  loads external assets). Use plain text, CSS, or inline SVG/data URIs.
- **Never use inline `style="..."` attributes or `<style>` blocks.**
  This is easy to get wrong, because it looks completely fine in local
  testing: this folder's real deployed response headers (see
  `vercel.json`) send `Content-Security-Policy: style-src 'self'`,
  which has no `'unsafe-inline'` and therefore silently blocks *every*
  inline style in production — the whole `<style>` block, or every
  `style="..."` attribute, just doesn't apply, with no error visible in
  the page itself. A plain local server (e.g. `python -m http.server`)
  doesn't send this header at all, so testing that way will not catch
  the problem — you have to either check against the real header or
  just never write an inline style in the first place. If an example
  needs its own one-off visual treatment (reproducing a specific real
  brand's look, say), add real classes to `example.css` instead, named
  for that example (e.g. `.cloud-alert`, `.survey-card`) so they're
  easy to find and don't collide with other examples' styling.

## Previewing while you write

Because every example links to `example.css` with a relative path and
has no dependency on the viewer page, you can open the file directly in
a browser tab while authoring it. Annotations will render in their
default (hidden) state — to check how they look with annotations on,
view the file through the actual viewer at `/resources/email-based`,
which is also the best way to confirm keyboard navigation, the
counter/title text, and the jump menu all pick up the new example
correctly.
