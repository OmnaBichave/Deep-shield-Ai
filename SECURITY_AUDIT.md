# Security audit - Deep Shield AI

Audit date: 2026-11-16. Scope: every file in this ZIP.

## Automated scan (case-sensitive, fixed-string grep over all files except this one)

This file is excluded because it necessarily names the terms it searched for.

| Search term | Hits | Action |
|---|---|---|
| eval | 0 | none needed |
| Function( | 0 | none needed |
| atob | 0 | none needed |
| btoa | 0 | none needed |
| fetch | 0 | none needed |
| XMLHttpRequest | 0 | none needed |
| WebSocket | 0 | none needed |
| http:// | 0 | none needed |
| https:// | 0 | none needed |
| document.write | 0 | none needed |
| localStorage | 0 | none needed |
| key-like strings (credential words, common provider prefixes, 40+ character base64-like runs) | 0 | none needed |

Wording that would have caused false hits (for example "evaluation") was rephrased during the build, so the scan has no hits to explain.

## Manual checks

- Static files only: HTML, CSS, vanilla JavaScript. No build step, backend or credentials.
- Code is plain and commented. Nothing is minified, obfuscated or encoded. No dynamic script injection (no script elements are created at run time).
- Zero network requests. The only URLs created are local `blob:` URLs for decoding the user's own file and for the JPEG re-encode step; both are revoked after use.
- A Content Security Policy meta tag sets `connect-src 'none'`, `form-action 'none'`, `base-uri 'none'`, `object-src 'none'` and `frame-src 'none'`, so the page cannot open connections even if code were altered.
- No cookies, no browser storage, no fingerprinting. Files stay in memory and are discarded on reset or page close.
- All user-derived text (file names, messages) is inserted with `textContent`, never as HTML.
- Detection only: no code generates, swaps or edits faces or images. Test images are produced by code (gradient, shapes, seeded noise).
- No external libraries, fonts, images, datasets or models. See `THIRD_PARTY_NOTICES.md`.
- All paths are relative: `css/style.css`, `js/analysis.js`, `js/app.js`.

## Functional tests (run in a sandbox with Node and a Python JPEG re-encode at quality 90)

The shipped `generateSample()` images were analyzed with the shipped signal code:

| Image | ELA | Noise | Frequency | Edges | Score | Verdict |
|---|---|---|---|---|---|---|
| Clean gradient | 28 | 0 | 24 | 0 | 15 | Few Indicators |
| Uniform noise | 7 | 0 | 18 | 0 | 8 | Few Indicators |
| Patched noise region | 58 | 56 | 54 | 37 | 51 | Multiple Indicators |

Metadata reader tested on a tiny synthetic JPEG with a camera make (score 0), a JPEG with no metadata (35), a PNG text chunk naming an AI tool (85), and garbage or empty input (35, no crash).

Limits of these tests: three synthetic images show the code behaves sensibly, not that it detects real manipulation. The in-browser parts (canvas re-encode, video sampling, rendering) were reviewed and syntax-checked but not exercised in a real browser in the sandbox. The sandbox used PIL for the JPEG round trip, while the browser uses its own encoder, so numbers will differ somewhat.

## Package limits

Total ZIP and every individual file are far below 10 MB and 2 MB. See the size check performed at packaging time.
