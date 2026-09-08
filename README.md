# ScamEducation.org

ScamEducation.org is a free educational resource designed to help individuals, families, and communities recognize, avoid, and respond to scams. The website provides accessible cybersecurity resources that explain common scam tactics, warning signs, and prevention strategies in plain language.

## About the Project

Scammers constantly evolve their methods, making it increasingly difficult to know what information, messages, and requests can be trusted. ScamEducation.org was created to make scam prevention information clear, accessible, and easy to use.

The project focuses on empowering users through education rather than fear or judgment. Our resources are designed to help people build confidence when identifying suspicious emails, phone calls, messages, and online interactions.

## Features

- Plain-language scam prevention resources
- Educational guides covering common scam tactics
- Printable materials for individuals, families, and community organizations
- A simulated phishing email inbox with 23 annotated, realistic examples covering gift cards, account alerts, sweepstakes, job offers, extortion attempts, and more
- Government reporting resources with direct links to the FTC, FBI IC3, CFPB, USPIS, SSA OIG, and state-level agencies
- A "You Scanned a QR Code" landing page for anyone who scans the QR code printed on our physical materials
- Interactive scam awareness activities
- Accessibility-focused design
- Resources suitable for users with varying levels of technical experience

## Project Structure

```
/
├── pages/
│   ├── index.html                       Home page
│   ├── about.html                       About ScamEducation.org
│   ├── licensing.html                   Licensing & attribution
│   └── resources/
│       ├── printable-guides.html        Downloadable PDF guides
│       ├── government.html              Federal & state reporting resources
│       ├── qr-code-scan.html            Landing page for printed-material QR codes
│       └── email-based/
│           ├── index.html               Simulated phishing email viewer
│           ├── manifest.json            Ordered list of examples shown in the viewer
│           └── phishing-examples/       23 individual simulated email examples
├── resources/
│   ├── *.pdf                            Printable guides served to visitors
│   └── government-icons/                Agency logos used on the Government Resources page
├── _archive/                            Retired pages kept for reference only (not linked from the live site)
├── vercel.json                          Routing, redirects, and security headers
└── README.md
```

Each page under `pages/` is a self-contained HTML document with its own inlined styles; there is no shared site-wide stylesheet for the main pages. The phishing examples share a single stylesheet (`phishing-examples/example.css`) and are rendered inside a sandboxed `<iframe>` by the viewer in `email-based/index.html`.

Adding a new phishing example: copy `phishing-examples/_TEMPLATE.html`, follow the instructions in its comments, then add an entry to `email-based/manifest.json` with a unique `id` (used as the page's `#anchor`), the `file` name, and the display `title`.

## Accessibility

Accessibility is a core part of ScamEducation.org's design. We want to make sure that our resources can be used by as many people as possible.

Accessibility considerations include:

- Clear, easy-to-understand language
- High-contrast design options
- Large, readable text
- Color-conscious design choices
- Resources designed for sharing and printing
- Skip-to-content links, labeled form controls, and descriptive `alt` text on every page
- Keyboard-focus styles and reduced-motion support
- Annotations in the phishing examples are conveyed through color, an underline, a numbered badge, and a written explanation together — never color alone — and interactive elements are linked to their explanation via `aria-describedby` so keyboard and screen-reader users get the same context as sighted users

## Security

Because this site displays realistic-looking phishing content for educational purposes, we take care to keep it inert and isolated:

- Every "malicious" link and button in the phishing examples is decorative only (`data-fake-url`) and has no real destination — nothing on the page can actually navigate a visitor anywhere
- Simulated emails render inside a sandboxed iframe with scripting disabled
- The phishing-examples directory is served with a locked-down Content Security Policy (`default-src 'none'`) that doesn't permit scripts at all
- Every route is served with a Content Security Policy, `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, and `Permissions-Policy` (see `vercel.json`)
- All outbound links use `rel="noopener noreferrer"`

## License

This project is provided for educational purposes. Please review ScamEducation.org/licensing for details regarding project usage and third-party resource attribution.

## Acknowledgments

ScamEducation.org uses icons and assets from third-party creators. Attribution information is available on ScamEducation.org/licensing.

Special thanks to the cybersecurity professionals, educators, and community members who support scam awareness and prevention.
