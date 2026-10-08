# SpeakStrong — Cloudflare migration runbook

Target window: 10–11 October 2026
Status: PREPARATION ONLY. No domain, DNS, repository visibility, or production deployment changes have been made.

## Current baseline (verified via GitHub, 8 October 2026)
- GitHub: AJM1975/table-topics (public), default branch main.
- Root: index.html, table-topics.html, questions.json, CNAME.
- CNAME contains speakstrong.app; existing deployment is GitHub Pages.
- table-topics.html fetches ./questions.json, requests browser microphone access using getUserMedia, and uses browser SpeechRecognition / webkitSpeechRecognition.
- Cloudflare Web Analytics beacon is already embedded in pages.
- Migration should preserve existing routes, content, question bank, and microphone permissions. Web Speech support remains browser-dependent.

## Goal
Move static website hosting to Cloudflare Pages, confirm the custom domain is working, then rename the repository to AJM1975/SpeakStrong and make it private. Keep site publicly available. Do not change live DNS before a passing staging test.

## Cloudflare Pages setup (manual account-authorized step)
1. In Cloudflare dashboard open Workers & Pages > Create application > Pages > Connect to Git.
2. Authorize Cloudflare Pages GitHub integration to access AJM1975/table-topics.
3. Set project name speakstrong (or another available name); choose main as production branch.
4. Build command: `exit 0` (or leave blank if UI allows); build output directory: `.` (repository root); root directory: `/`.
5. Deploy and record the exact *.pages.dev URL. Review Cloudflare deployment output.
6. Confirm the preview works before attaching the domain. Test on desktop and mobile.

## Smoke tests — must pass before DNS cutover
- [ ] Root / landing page loads with correct styling and navigation.
- [ ] /table-topics.html loads and displays initial question prompt.
- [ ] /questions.json is valid JSON served with success status.
- [ ] Generate new question; difficulty selection and repeat avoidance behave as before.
- [ ] Timer starts, stops and resets.
- [ ] Microphone permission prompt appears over HTTPS.
- [ ] Audio recording and playback work in supported browsers.
- [ ] Speech transcription works where SpeechRecognition is supported; unsupported browsers show graceful guidance.
- [ ] Rubric scoring/evaluation still functions.
- [ ] Mobile viewport and controls are usable.
- [ ] Cloudflare Analytics events are observed; no double counting beacons.
- [ ] Direct navigation and refresh to both HTML paths work.

## Domain cutover — AFTER tests
1. In the Pages project, add custom domain speakstrong.app.
2. Check where authoritative DNS is actually hosted (do not assume registrar Porkbun controls DNS); follow Cloudflare's domain onboarding and DNS verification instructions for the exact setup.
3. Record pre-cutover DNS values and TTL for rollback.
4. Update only necessary DNS records, confirm certificate/HTTPS issuance and both apex/www behavior (if www is desired).
5. Test speakstrong.app live including questions.json and recording. Monitor errors and analytics.

## Repository privacy — LAST
1. In GitHub Settings > General, rename table-topics to SpeakStrong.
2. Confirm Cloudflare Git integration points to renamed repo and a test commit redeploys.
3. In GitHub Settings > General > Danger Zone, change repository visibility to Private.
4. Verify Cloudflare has GitHub App access to the private repo and an actual deploy succeeds.
5. Disable or retire prior GitHub Pages publishing only once Cloudflare production is verified.
6. Note that privacy does not erase public forks, historical clones, or cached copies.

## Rollback
- If staging fails, no cutover; GitHub Pages remains active.
- If production cutover fails, restore recorded DNS values, allow DNS caches to expire, and verify the existing GitHub Pages destination still works.
- Avoid making repository private before successful Cloudflare cutover because GitHub Pages availability may change.
- Preserve the original main branch baseline and make all changes through PRs.

## Release scope rules
- In: infrastructure migration, smoke tests, analytics verification, documentation, rollback.
- Out: UI redesign, databases, APIs, new evaluation features, auth/user accounts.

## Open decisions
- Exact Cloudflare account/project selected and whether speakstrong.app zone is already managed in Cloudflare.
- Current DNS records and authoritative nameservers.
- Desired www redirect policy.
- Project Hub's latest backlog: cannot be assumed until live items are obtained.

References:
- https://developers.cloudflare.com/pages/framework-guides/deploy-anything/
- https://developers.cloudflare.com/pages/configuration/git-integration/
