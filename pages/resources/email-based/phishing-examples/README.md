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

Wrap the exact text or element that gives the scam away like this:

```html
<mark class="annotation">paypa1-secure-center.com
  <span class="annotation-note">
    This isn't a PayPal domain — legitimate PayPal emails come from
    an address ending in @paypal.com.
  </span>
</mark>
```

That's the whole convention:

- `mark.annotation` wraps the suspicious text itself.
- A nested `span.annotation-note` holds the plain-language explanation.
- Numbering, highlighting, and show/hide behavior are all handled by
  `example.css` automatically — never add a number, an id, or any
  JavaScript yourself.

Because numbering is generated with a CSS counter, you can add, remove,
or reorder annotations freely and they'll always renumber correctly.

A few practical notes:

- Annotate the smallest span of text that makes the point — a domain
  name, not the whole sentence around it — *unless* the red flag is the
  sentence as a whole (e.g. an urgency/threat line), in which case
  wrapping the full sentence is clearer.
- To annotate a button or link, wrap the whole `<a>` in the `mark`
  rather than putting the `mark` inside the `<a>` — this makes the
  explanation render as a box below the button instead of being
  squeezed inside it:

  ```html
  <mark class="annotation">
    <a href="#" class="email-btn">Verify My Account</a>
    <span class="annotation-note">Explanation here.</span>
  </mark>
  ```
- Keep explanations short — one or two sentences. They need to be
  readable at a glance on a phone screen.
- If the flagged text is immediately followed by trailing punctuation
  that's *not* part of the annotation — like the closing `>` after a
  sender's email address — wrap it in `<span class="annotation-suffix">`
  and put that **inside** the `mark`, right after the flagged text and
  before the `annotation-note` span, rather than after the closing
  `</mark>`:

  ```html
  <!-- do this -->
  <mark class="annotation">suspicious-domain.com<span class="annotation-suffix">&gt;</span><span class="annotation-note">...</span></mark>

  <!-- not this -->
  <mark class="annotation">suspicious-domain.com<span class="annotation-note">...</span></mark>&gt;
  ```

  `annotation-note` is a block-level element, so anything placed after
  the closing `</mark>` gets pushed onto its own line below the note
  box instead of staying attached to the text before it — a lone `>`
  stranded on its own line is the visible symptom. `annotation-suffix`
  keeps it glued to the flagged text on the original line without
  picking up the highlight or dotted underline, since it isn't actually
  part of what's flagged. It currently assumes a meta-header (light
  gray) background — if you ever need it somewhere with a different
  background, add a matching override in `example.css`.
- Keep every `mark.annotation` on a single line with no whitespace
  between its child tags (including before the closing `</mark>`).
  `annotation-note` is block-level, so a stray whitespace-only text
  node left dangling after it (easy to introduce by indenting a button
  annotation across several lines) renders as its own thin highlighted
  blank line below the note box.

## Security model

These are simulated phishing emails on a public education site, so this
folder is written under the assumption that a mistake here should never
be able to do anything harmful. Several layers enforce that independently:

1. **The viewer loads every example in a sandboxed `<iframe>`**
   (`sandbox="allow-same-origin"`, deliberately *without* `allow-scripts`,
   `allow-forms`, `allow-popups`, or `allow-top-navigation`). This is the
   primary control — it's enforced by the browser and doesn't depend on
   anything in this folder being written correctly.
2. **`example.css` makes every link inert** (`pointer-events: none`) as a
   second, independent layer, in case a future example is authored with
   a real `href` by mistake.
3. **Response headers** (see the root `vercel.json`) send a strict
   `Content-Security-Policy` for everything under this folder, blocking
   scripts and form submissions at the network level as a third layer.

Given that, when writing an example:

- **Never add a `<script>` tag.** It will not execute, and its presence
  would be misleading to a future reader of the source.
- **Never use a real URL.** Always use `href="#"` for any link or
  button. Don't rely on the sandbox alone — write it correctly.
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
