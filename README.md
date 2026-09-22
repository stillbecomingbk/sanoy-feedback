# sanoy-feedback

An ongoing feedback system for Sanoy to keep a tab on its customers and preferences.

    the form      https://stillbecomingbk.github.io/sanoy-feedback/
    reading it    https://stillbecomingbk.github.io/sanoy-feedback/review.html

Static site. No build step, no dependencies to install.

    index.html        the form: markup, app and all six translations
    review.html       private dashboard — Google sign-in, search, CSV export
    runtime.js        Claude Design runtime, extracted from the original bundle
    vendor/           React 18.3.1 + React-DOM, pinned locally (no CDN)
    assets/           images and the two webfont families

Images and fonts are separate files on purpose. They were inlined as data
URIs at one point, which fails: the runtime writes backgrounds into inline
`style` attributes and splits those on `;`, so `data:image/jpeg;base64,...`
is cut at the semicolon and the image disappears.

## Pages

Settings → Pages → Source: **Deploy from a branch** → `main` → `/ (root)`.

Custom domain: add a file named `CNAME` at the root containing
`feedback.sanoycare.com`, then point a CNAME record at `stillbecomingbk.github.io`.
A new domain must also be added under Firebase → Authentication → Settings →
Authorized domains, or sign-in on the review page will refuse to open.

## Languages

English, Hindi, Telugu, French, Spanish, Russian — complete UI in each.
Product names, sizes and the company name stay in English by design.
Each non-English version carries a governing-language note: the English
text prevails in case of discrepancy.

## One link per channel

This form is the only way Sanoy collects feedback — online, in stores,
sampling, events — so every answer records where it came from. Add tags to
the link; each lands in the answer as `Source — …` (and in the CSV):

    ?src=event&event=mumbai-popup&by=priya     an event, and who ran the stand
    ?src=store&event=hyderabad-jubilee         a store
    ?src=sampling&event=oct-sample-box         a sampling drop
    ?src=instagram                             a post or story
    ?src=email&event=8-week                    the follow-up email
    ?utm_source=…&utm_medium=…&utm_campaign=…  also understood

No tag = `direct`. Make a QR code from the tagged link for print.

**Kiosk mode** — add `&kiosk=1` for a tablet on a stand: after each answer
the form clears itself (30 seconds after the thank-you) for the next person.

**No connection** — if the device is offline when someone presses send, the
answer is kept on that device and sent automatically when the connection
returns (next load, or the moment it comes back online). A refusal from the
database is never hidden this way; only network failures are queued. Leave
the kiosk tab open until it has been online again.

Every answer gets its own reference (`SNY-F` + time + random); the old
reference was worked out from the answers and two people could share one.

## Where the answers go

Firestore, in the Firebase project `sanoy-feedback` (Mumbai region), as one
document per response in the `responses` collection.

This started as a Google Apps Script web app writing to a Sheet, and that
cannot be made to work on a personal Google account: Apps Script has to ask
for `spreadsheets` and `drive` — scopes Google now classes as sensitive —
and it hard-blocks unverified apps that request them, with no way for the
owner to approve it. Firestore's REST API is the path meant for a public
web client: a browser key that identifies the project and grants nothing,
with security rules doing the actual work.

The rules, in `firestore.rules` terms:

    create   anyone, if the document has the expected shape
    read     only bhanumeraki@gmail.com, signed in with Google
    update   nobody, ever
    delete   nobody, ever

Append-only is deliberate. A response is a record of what somebody actually
said, and publishing a quote means standing behind it.

Each document holds a few top-level fields for sorting and searching
(`reference`, `submittedAt`, `language`, `overall`, `products`) plus a
`record` map carrying every answer keyed by its own question. No two
submissions share a field set — the form asks about whichever products
that person used — and a map handles that without any schema to migrate.
The channel tags live inside `record` for that reason: no rule change.

Photographs ride inside the document. Firebase Storage needs a paid plan,
and Firestore allows 1 MiB per document, so the browser downscales to a
1400 px longest edge at quality 0.78 — roughly 240 KB once base64-encoded,
which leaves ample room. A 6 MB phone photograph arrives comfortably.

## Reading responses

`review.html` signs in with Google and lists everything, newest first, with
search across all written answers and a CSV download. Each card shows the
channel and event beside the reference. The CSV carries a UTF-8 BOM so Excel
opens the Hindi, Telugu and Russian responses as text rather than mojibake.

Everything is on the Firebase free tier. There is no card on the account.
