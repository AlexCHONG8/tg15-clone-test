# T.G. 15 Mirror — Clone Test

Static mirror of the Indufar (Paraguay) vademecum product page for **T.G. 15 (Tirzepatida 15 mg/0.5 mL)**, captured 2026-10-20 for a clone-tooling quality test.

- Original: https://www.indufarverify.com/web/vademecum/productos/detalles/400002533.html
- Sibling doses: T.G. 5 = 400002516, T.G. 7.5 = 400002525, T.G. 10 = 400002526, T.G. 12.5 = 400002527, T.G. 15 = 400002533
- Method: raw HTML via curl + asset crawl (CSS/JS/fonts/images referenced by HTML, CSS, and runtime network log). Absolute server paths preserved so `index.html` serves unmodified from repo root.
- 56 files, ~7 MB. Google Fonts (Roboto, Open Sans, Material Icons) load remotely; all other assets are local.
- Verified: loads in Chrome with zero console errors; layout, fonts, images, and footer banner match original.

## REA test result

[REA](https://github.com/morluto/rea) (npm `rea-agents` 6.7.0) was tested against the live page via CDP (`rea list-browser-targets` → `rea analyze-web-bundle` → `rea capture-web-screenshot`).

Verdict: **REA does not duplicate websites.** It is a binary/app reverse-engineering suite (Ghidra/Hopper/IDA, Android/iOS/harmony, JS bundle static analysis). For this web target it produced:

- Script inventory with SHA-256 digests (jquery.js, inline page script, app.min.js, moment-with-locales.min.js)
- One import edge (app.min.js requires jquery), one route (`/`)
- 8 low-confidence "endpoint candidates" — all false positives from `Cookies.get` string args
- A 756x413 viewport screenshot (worked)

It cannot reconstruct HTML/CSS or produce a runnable copy. The mirror here was built with plain curl + asset crawl, not REA. REA is the right tool for understanding compiled/bundled app internals, not for cloning public pages.

Test artifacts: `rea-bundle-analysis.json` (REA evidence), `rea-original-screenshot.png` (REA capture of the original).

## Mock verify page

`customer/verificar.html` — mobile verify card (Indufar styling) reading `?lote=` (default 66903; parser tolerant of appended `?coord=…&useClipApp=true` junk from the AuthNow app's verbatim `/customer/` URL branch). `customer/qr-66903.png` — QR (error-correction H) encoding `https://alexchong8.github.io/tg15-clone-test/customer/verificar.html?lote=66903`. AuthNow APK REA findings live one level up in `../authnow-rea-findings.md`.

## 2026-10-21 scan fixes

- `customer/qr.html` — NEW phone-side QR target: inline SVG QR (viewBox-correct, error-correction H) sized to `min(100vw-32px, 78vh)` so modules render large and crisp on a phone screen. Root cause of the failed Mac-camera scan: a plain PNG opened in a phone browser renders the QR ~120–180 px wide → Mac webcam capture too small/moiré-affected for jsQR.
- `customer/welcome.html` — scanner hardening: persistent status pill (`Buscando código…` / `Código detectado…` + capture resolution), local vendored `customer/jsQR.min.js` (CDN only as fallback), `Clave manual` button (camera-free entry), and `?lote=` deep link → jumps straight to the verify card.
- Verified headless (Playwright + fake camera fed from a 390×844 screenshot of qr.html): decode succeeds with QR at 340/240/160 px in a 640×480 frame.
