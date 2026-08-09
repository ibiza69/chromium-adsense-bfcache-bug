# chromium-adsense-bfcache-bug
Chromium mobile back/forward cache bug (Blink) freezing Google AdSense scripts and web layouts, causing dynamic text stalls and stuck vignette hashes.

## Technical Incident Report: Chromium Bfcache Restoration of Stale AdSense Vignette State

### Bug Reference Data
- **Chromium Bug Tracker ID:** 544217146
- **Affected Component:** `Blink>History>BackForwardCache`
- **Impacted Services:** Google AdSense (Mobile Vignette Ads), Core Web Vitals (INP, CLS), Dynamic Site Scripts.
- **AdSense Support Thread:** https://support.google.com/adsense/thread/447696714/bug-doble-en-intersticiales-bfcache-aria-hidden-rompe-anuncios-m%C3%B3viles?hl=es


### Real Bug Mechanism & Technical Breakdown
There is a fundamental state-restoration flaw when a user navigates away from a page via an AdSense mobile vignette link and subsequently triggers a Back/Forward Cache (bfcache) hit in modern mobile Chromium.

1. **Initial Trigger:** Clicking a dynamic element (e.g., a job posting link) initializes the AdSense vignette overlay. At this precise point, the console triggers a red warning: *"Blocked aria-hidden on a <body> element because it would hide the entire accessibility tree"*. 
2. **Navigation:** Upon closing/progressing past the ad, the DOM is correctly cleaned up, the `aria-hidden` attribute disappears from the `<body>`, and the user is redirected to a completely clean internal destination URL. No further console errors are thrown.
3. **The Deadlock on Back Navigation:** When the user clicks the browser's "Back" button, Chromium bypasses a clean network reload and fetches the previous page state directly from the `bfcache` snapshot. 

**The Root Cause Failure:** Chromium caches the state of the homepage *prior* to the complete programmatic teardown of the vignette infrastructure. Upon restoration, the homepage wakes up entirely frozen. The browser address bar remains permanently hijacked with the uncleaned `/#google_vignette` hash, dynamic text items/feeds stall without fetching updates, and all standard responsive and sticky ad units freeze in place, generating an absolute drop in mobile RPM impressions.

---

### Status and Publisher Blockade
Official Sellside engineering channels have privately confirmed that their monetization scripts were architected under a hard "NO BFCACHE" assumption. They have classified this systemic failure as a "long-term" problem. 

Publishers are currently left in a complete deadlock: attempting to patch the layout or forcefully reload the window via automated script manipulation directly breaches Google AdSense policies regarding ad code alteration and invalid traffic generation, leading to immediate automated account bans. Leaving the code unmodified forces the mobile UI into absolute zero-responsiveness, destroying real-world user metrics and mobile ad impressions.
