# Phishing examples

This folder holds the individual simulated phishing emails shown by the
viewer at `/resources/email-based` (`../index.html`). Each example is its
own plain HTML file, so examples can be added, edited, or removed one at a
time without touching the viewer.

## Adding a new example

1. Copy `_TEMPLATE.html` to a new file in this folder, e.g.
   `03-fake-gift-card.html`. Numbering the file is a convenience for
   humans browsing the folder — the actual display order comes from
   `manifest.json`, not the filename.
2. Write the email: subject, from/to/date, and body, following the
   pattern already in the template and in `01-paypal-verification.html`.
3. Wrap each phishing indicator in an annotation (see below).
4. Add an entry to `../manifest.json`:

   ```json
   {
     "id": "fake-gift-card",
     "file": "03-fake-gift-card.html",
     "title": "Fake Gift Card Request",
     "description": "One sentence describing the scam for the viewer's UI."
   }
   ```

   `id` should be a short, unique, URL-safe slug — it's used for
   deep-linking to a specific example (`/resources/email-based#fake-gift-card`).

That's the entire process. The viewer doesn't need any code changes to
pick up a new example — it reads the manifest at load time.

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
  need to be nested inside anything to do that). Every link/button
  needs a `data-fake-url` attribute — hovering or focusing it shows
  that as a fake destination, the same way a real email lets you check
  where a link actually goes before clicking:

  ```html
  <mark class="annotation" data-color="2"><a href="#" class="email-btn" data-fake-url="https://bash.video">Verify My Account</a></mark><span class="annotation-note" data-color="2">Explanation here.</span>
  ```

  Leave off `class="email-btn"` for a plain inline text link instead of
  a styled button — the base link style (blue, underlined) applies
  automatically. **Don't make every example a "click the big button"
  scam.** Real phishing just as often asks you to reply, call a number,
  wire money, or hand over gift card codes with no link at all — vary
  it so the lesson isn't "buttons are the tell," it's "read carefully."
  A rough mix across all the examples: some with a styled button, some
  with only a plain text link, some with neither.
- Keep explanations short — one or two sentences. They need to be
  readable at a glance on a phone screen.
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
2. **Every link's real target is always `href="#"`**, which only jumps
   within the current sandboxed document — it can never navigate away
   from the page or open anything, regardless of what a link's fake
   preview text says. Links are no longer `pointer-events: none` (they
   used to be) — that's what made the hover-preview feature possible,
   since a browser won't show `:hover` state at all on an element with
   `pointer-events: none`. Removing it doesn't weaken anything: `href="#"`
   was always the actual safety boundary, not the inability to hover.
3. **Response headers** (see the root `vercel.json`) send a strict
   `Content-Security-Policy` for everything under this folder, blocking
   scripts and form submissions at the network level as a third layer.

Given that, when writing an example:

- **Never add a `<script>` tag.** It will not execute, and its presence
  would be misleading to a future reader of the source.
- **Never use a real URL.** Always use `href="#"` for any link or
  button — don't rely on the sandbox alone, write it correctly. If you
  want to show a fake destination on hover (recommended — see the
  annotation convention above), put it in `data-fake-url`, never in
  `href` itself.
- **Don't add forms.** If an example needs to show a fake login form,
  build it with plain `<input>`/`<label>` markup and no `<form>` tag or
  `action`/`method` attributes.
- **Don't reference external images or fonts.** Keep every example
  self-contained (this also matches the rest of the site, which never
  loads external assets). Use plain text, CSS, or inline SVG.

## Previewing while you write

Because every example links to `example.css` with a relative path and
has no dependency on the viewer page, you can open the file directly in
a browser tab while authoring it. Annotations will render in their
default (hidden) state — to check how they look with annotations on,
view the file through the actual viewer at `/resources/email-based`,
which is also the best way to confirm keyboard navigation, the
counter/title text, and the jump menu all pick up the new example
correctly.
