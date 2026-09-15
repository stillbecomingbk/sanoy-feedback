# sanoy-feedback

An ongoing feedback system for Sanoy to keep a tab on its customers and preferences.

Live at https://stillbecomingbk.github.io/sanoy-feedback/

Static site. No build step, no dependencies to install.

    index.html        the form: markup, app and all six translations
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

## Languages

English, Hindi, Telugu, French, Spanish, Russian — complete UI in each.
Product names, sizes and the company name stay in English by design.
Each non-English version carries a governing-language note: the English
text prevails in case of discrepancy.

## Where the answers go

`Code.gs` (supplied separately) is a Google Apps Script web app that writes
each submission as a row in a Google Sheet and each photograph into a Drive
folder. Deploy it as a web app (Execute as: Me, Who has access: Anyone),
then put the resulting URL into `index.html`:

    window.SANOY_ENDPOINT = 'https://script.google.com/macros/s/..../exec';

Until that is set the form still works and still thanks the respondent —
it simply has nowhere to write.

The sheet grows its own header row: each respondent answers about whichever
products they actually used, so no two submissions carry the same field set.
Columns are created on first sight and matched by name after that.

Photographs are downscaled in the browser to a 1600 px longest edge before
they are sent, so a 6 MB phone photograph arrives as roughly 250 KB.
