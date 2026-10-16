# napplet.link

The static site behind `https://napplet.link`, served by Cloudflare Pages.

## What it does

- **Android App Links.** `/.well-known/assetlinks.json` vouches for the Myco
  app (`app.myco`), so Android opens `https://napplet.link/...` links in Myco
  straight away instead of leaving "Open supported links" off.
- **Browser fallback.** Any `/<archetype>/<intent>?…` path shows a small page
  with an "Open in your napplet shell" button (`napplet:<archetype>/<intent>?…`).
  `404.html` is the same page, so every deep link renders.

## Signing certificates

`sha256_cert_fingerprints` must list every certificate Myco is signed with:

- the debug key on the dev Mac (in the file now);
- the release key (`myco-release.jks`) — add its fingerprint before a release:

  ```
  keytool -list -v -keystore myco-release.jks -alias <alias> | grep SHA256
  ```

  Play App Signing re-signs with Google's key: add that one too (Play Console →
  App integrity), if Myco ever ships there.

## Deploy (Cloudflare Pages, from GitHub)

1. Push this repo to GitHub.
2. Cloudflare dashboard → Workers & Pages → Create → Pages → Connect to Git →
   pick the repo. Framework preset: **None**. Build command: *empty*. Build
   output directory: `/`.
3. After the first deploy: the project → Custom domains → add `napplet.link`
   (the zone must be on Cloudflare; it adds the DNS record).
4. Check it: `curl -i https://napplet.link/.well-known/assetlinks.json` must
   answer **200**, `content-type: application/json`, with **no redirect**.
5. On a phone with Myco installed (or reinstalled after the file is live):
   `adb shell pm verify-app-links --re-verify app.myco`, then
   `adb shell pm get-app-links app.myco` should show `napplet.link: verified`.

Every push to `main` redeploys.
