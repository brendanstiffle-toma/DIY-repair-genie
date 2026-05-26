# DIY Mechanic Genie — Design Sketch

A mobile-first web app (PWA) that turns "something's wrong with my car" into a confident decision: fix it yourself with the right parts and a trustworthy video, or hand it to a shop with a fair quote in your pocket.

Target: runs in iOS Safari (no App Store), built as a Next.js PWA so it can later be "Add to Home Screen" with near-native feel.

---

## 1. Product positioning

The strongest moat isn't any individual integration — VIN decoders, parts catalogs, and YouTube are commodities. The moat is the **matching layer**: turning a photo of a torn CV boot plus "clicking noise on left turns" plus a 2014 Subaru Outback into:

- the right diagnosis (with calibrated confidence)
- the exact part number (and three places to buy it at known prices)
- the two or three actually-good YouTube tutorials for *this* year/make/model (not just the make in general)
- an honest "DIY in 2 hours with $80 in parts" vs "$450 at a shop" comparison
- the names of two local shops who are good at this specific job

Everything else in the app is plumbing to make that matching layer feel magical.

---

## 2. End-to-end user flow

Twelve screens, every one tappable on the mockup:

1. **Welcome** — single CTA: "Start a diagnosis."
2. **Vehicle ID method** — three choices: scan VIN, type VIN, pick year/make/model manually.
3. **VIN camera** — live camera with a guide rectangle over the VIN area. OCR runs on each frame; when 17 valid chars lock, auto-advance.
4. **Vehicle confirm** — pre-filled Year / Make / Model / Engine / Trim dropdowns from NHTSA decode; user can correct anything.
5. **Problem capture** — two inputs: (a) optional photo of the symptom (broken part, fluid leak, dashboard light), (b) text or voice describing what's happening ("clicking when I turn left at low speed").
6. **Clarifying questions** — Claude asks 1–3 targeted follow-ups to narrow the diagnosis (e.g., "Does the clicking happen only when turning, or also going straight?").
7. **Diagnosis** — ranked list of likely causes with confidence percentages and a short plain-English explanation of each.
8. **Job overview** — for the top diagnosis: difficulty (1–5 wrench icons), estimated DIY time, parts cost range, tools needed.
9. **Parts** — exact OEM and aftermarket options with live pricing from eBay/Amazon/RockAuto, sortable by price/rating/shipping.
10. **Tools** — checklist of required tools; flag which ones the user likely already owns vs needs to buy/borrow; AutoZone loan-a-tool eligibility.
11. **DIY guide** — top 2–3 YouTube videos curated for this exact YMM + job, plus the best Reddit threads (often more honest about the gotchas). Watch inline.
12. **DIY vs Shop** — side-by-side card: total DIY cost + time vs estimated shop quote (RepairPal range), then "Find a shop" button.
13. **Nearby shops** — 3–5 shops within 10 miles, ranked by specialization for this YMM/job, with ratings, prices, and "Call" / "Get directions" buttons.

Reverse flow is also allowed: a power user can skip diagnosis and jump straight to "I know I need brake pads" → directly to the parts step.

---

## 2a. The job checklist (persistent across the flow)

A checklist lives at the top of every screen as a thin status strip the user can tap to expand. It is the spine of the experience — the thing that turns this from "an answer" into "a project I'm tracking."

**Phase A — Diagnosis (always present, populates as the user moves through screens):**

- [ ] Vehicle identified
- [ ] Problem described
- [ ] Diagnosis confirmed

**Phase B — Path chosen (appears after the user taps "I'll DIY this" or "Get a shop quote" on screen 12):**

If DIY:

- [ ] Parts ordered
- [ ] Tools gathered (rented / borrowed / bought)
- [ ] Parts received
- [ ] Workspace prepared (jack stands, drain pan, etc.)
- [ ] Repair started
- [ ] Repair complete
- [ ] Test drive completed
- [ ] Feedback logged (rate the diagnosis + the videos)

If Shop:

- [ ] Quote requested from shop
- [ ] Appointment scheduled
- [ ] Vehicle dropped off
- [ ] Service complete
- [ ] Vehicle picked up
- [ ] Test drive completed
- [ ] Feedback logged (rate the shop + the diagnosis)

Items are auto-checked when we can detect the event (e.g., "Parts ordered" flips when the user taps the eBay buy link), and tap-to-check for everything else. The checklist persists in localStorage so the user can close the tab and come back.

A small banner reminds the user of their next open item: *"Next: check that parts arrived → tap when they're here."*

---

## 3. Tech stack

| Layer | Choice | Why |
|---|---|---|
| Framework | **Next.js 14 (App Router)** deployed on Vercel | Fast iteration, API routes for backend, easy PWA, free tier |
| UI | React + Tailwind + shadcn/ui | iOS-feeling components without writing CSS from scratch |
| State | React Server Components + Zustand for client | Simple, no Redux overhead |
| Camera | `navigator.mediaDevices.getUserMedia` | Standard Web API; works in iOS Safari 14.3+ |
| VIN OCR | **Tesseract.js** in a Web Worker for client-side, with a server fallback that sends the frame to Claude Vision if local OCR fails twice | Client-side is fast and private; Claude is the safety net |
| Geolocation | `navigator.geolocation` | Standard; iOS prompts the user |
| Voice input | `webkitSpeechRecognition` (iOS Safari supports it) | Hands-free symptom description |
| Backend | Next.js Route Handlers | One repo, one deploy |
| AI | **Anthropic Claude (claude-sonnet-4-6)** for vision + reasoning + diagnosis JSON | Strong vision, strong structured output, fits Toma stack |
| DB | Postgres on Supabase (free tier) | Stores user diagnoses, feedback, shop reviews |
| Auth | Supabase Auth or none (anon for MVP) | Skip auth for prototype |
| Analytics | PostHog | Track funnel drop-off per screen |

iOS Safari constraints to design around:

- Camera requires HTTPS — Vercel gives this free.
- `getUserMedia` will not work in an iframe unless the parent grants permission — keep the app top-level.
- iOS rejects autoplay video with sound; mute the video element used for the camera preview.
- "Add to Home Screen" PWA does not get push notifications until iOS 16.4+, and only when launched from the home screen icon — fine for MVP.
- Safe-area insets matter — use `env(safe-area-inset-*)` for the camera and bottom CTAs.

---

## 4. External APIs and what they cost

| API | What it gives us | Cost | Notes |
|---|---|---|---|
| **NHTSA vPIC** | VIN → year/make/model/engine/trim | Free, no key | The right backbone. Endpoint: `vpic.nhtsa.dot.gov/api/vehicles/DecodeVin/{VIN}?format=json` |
| **Anthropic Claude API** | Diagnosis reasoning, photo understanding, follow-up question generation, YouTube/Reddit result re-ranking | Per-token | Use Sonnet 4.6 for diagnosis, Haiku 4.5 for cheap re-ranking |
| **eBay Browse API** | Real parts pricing, OEM and aftermarket, by VIN-compatible part fitment | Free tier with key | Has a `compatibility` filter — pass YMM and get only parts that fit |
| **Amazon Product Advertising API (PA-API 5)** | Cross-check parts pricing on Amazon | Free, requires Amazon Associates approval (commission flows to us) | Affiliate revenue path |
| **RockAuto** | The DIY canon for parts | No official API — scraping is fragile and against ToS | Skip for MVP; link out instead |
| **AutoZone** | Parts + loan-a-tool | No public API | Link out + deep-link to their store locator |
| **YouTube Data API v3** | Find tutorial videos | Free up to 10,000 quota units/day; a search = 100 units | Cache aggressively per YMM+job |
| **Reddit API** | Find threads from r/MechanicAdvice, r/Cartalk, vehicle-specific subs | Free at low volume; OAuth required | We're not building a Reddit client; we're surfacing 1–3 top threads per diagnosis |
| **Google Places API** | Nearby auto repair shops | Pay-per-call after free $200/mo credit | Acceptable for MVP scale |
| **RepairPal Estimator** | Shop quote ranges by YMM + job + ZIP | Paid B2B API | License once we have product-market fit; for MVP, use a rough labor-hours × $130/hr heuristic |

---

## 5. The matching layer (the actual product)

This is where Claude earns its keep. The diagnosis call looks roughly like this:

```text
System: You are a master mechanic. Diagnose the user's car problem.
Always respond with JSON matching the schema.

User: {
  vehicle: { year: 2014, make: "Subaru", model: "Outback",
             engine: "2.5L H4", trim: "2.5i Premium" },
  photo: [image of torn rubber boot on driver-side axle],
  symptoms: "clicking noise when turning left at low speed,
             worse when accelerating out of the turn",
  followup_answers: { ... }
}

Response schema:
{
  diagnoses: [
    { cause: string, confidence: 0..1, reasoning: string,
      parts: [{ name, oem_part_number, alternatives }],
      tools: [{ name, common_to_own: bool, rentable_at_autozone: bool }],
      difficulty: 1..5, est_diy_minutes: int,
      est_shop_dollars: { low, high },
      youtube_search_terms: string,
      reddit_search_terms: string,
      diy_warnings: [string]
    }
  ]
}
```

The structured JSON drives every downstream screen. The parts step takes `parts[].oem_part_number` and queries eBay; the videos step takes `youtube_search_terms` and calls YouTube; etc. No screen except diagnosis talks to Claude directly — that keeps token costs predictable.

A second cheap Haiku call re-ranks YouTube results: feed the title + description + view count of the top 10 results and ask "which 3 of these are actually walking through this exact job for a 2014 Subaru Outback?" View count alone overweights popular but generic content.

---

## 6. Data model (minimal)

```
diagnoses
  id, user_id (nullable for anon), created_at
  vehicle_jsonb     -- VIN decode result
  symptoms_text
  photo_url (Supabase storage)
  claude_response_jsonb
  final_action_taken  -- "diy" | "shop" | "abandoned"
  feedback (1..5 stars, optional)

parts_lookups
  id, diagnosis_id, oem_part_number,
  ebay_price_cents, amazon_price_cents, fetched_at

shop_clicks
  diagnosis_id, shop_place_id, clicked_at

job_checklists
  id, diagnosis_id, path  -- "diy" | "shop"
  items_jsonb             -- [{key, label, checked_at, auto_detected}]
  created_at, updated_at
```

That's it for MVP. Feedback on the diagnosis is the most valuable data — it's what lets us improve the matching layer over time.

---

## 7. Build path

| Phase | Scope | Time |
|---|---|---|
| **Phase 0** | This design + clickable HTML mockup (already done) | done |
| **Phase 1** | Next.js skeleton, camera screen, NHTSA VIN decode wired up, vehicle confirm screen | 1 day |
| **Phase 2** | Problem capture + Claude diagnosis call + diagnosis display | 1–2 days |
| **Phase 3** | eBay Browse API integration for real parts pricing; YouTube + Reddit surfacing | 2 days |
| **Phase 4** | Google Places nearby shops + DIY-vs-shop comparison | 1 day |
| **Phase 5** | Polish, PWA manifest, "Add to Home Screen" prompt, basic analytics | 1 day |

A working prototype is ~5–7 focused days for one developer.

---

## 8. Risks and tradeoffs

- **Liability** — telling a user "you can DIY this brake job" creates exposure if they get hurt. Mitigation: ship a clear disclaimer, never advise on safety-critical work (airbags, fuel system) without strong language directing them to a shop, and flag jobs that require torque specs or alignment.
- **Hallucinated part numbers** — Claude will sometimes invent OEM part numbers. Mitigation: validate every part number against eBay/Amazon fitment results before showing to user. If no match, hide the part number and show only the part name.
- **Stale YouTube ranking** — view count favors viral over correct. Mitigation: the Haiku re-rank step above.
- **iOS Safari camera quirks** — front-facing camera by default unless `facingMode: "environment"` is set, no torch control in Safari (helpful for dim VIN plates in door jambs). Mitigation: default to back camera and provide a "tap to upload from camera roll" fallback.
- **eBay fitment data quality** — for older vehicles (pre-2000) it's spotty. Mitigation: warn user, fall back to make-only search.

---

## 9. What's intentionally *not* in the MVP

- User accounts and saved vehicles
- Maintenance schedule / reminder system
- Recall lookups (NHTSA has a free API for this — easy v1.1 add)
- A marketplace where local mechanics bid on the job
- AR overlay showing where a part is located in the engine bay (cool, but a Phase 3 idea)

---

## 10. Open questions for next iteration

1. Affiliate revenue (eBay Partner Network, Amazon Associates) vs SaaS subscription vs lead-gen to shops — which to optimize for first?
2. Do we capture VIN once and remember the vehicle (requires auth), or stay anonymous and re-scan every time?
3. Voice-first interface — should the entire flow be doable hands-free while the user is under the hood? Speech-to-text is there, text-to-speech is there, just a UX question.
