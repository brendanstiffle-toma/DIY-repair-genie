# DIY Mechanic Genie

A mobile-first web app concept that turns "something's wrong with my car" into a confident decision: fix it yourself with the right parts and a trustworthy video, or hand it to a shop with a fair quote in your pocket.

**[Try the clickable mockup →](https://brendanstiffle-toma.github.io/diy-mechanic-genie/)**

> Status: early prototype, mocked data, looking for feedback before building the real thing.

> **Who built this:** I'm Brendan at [Toma](https://toma.com). Toma builds AI for the automotive industry — today we work with dealers and enterprise customers. This mockup is an exploration of whether a direct-to-consumer product makes sense in the same space. Posting publicly because honest outside feedback is the cheapest way to find out.

---

## What this is

A clickable HTML mockup of every screen in the user flow — VIN scan, vehicle confirmation, problem capture, AI diagnosis, parts list, tools, curated YouTube + Reddit guides, DIY-vs-shop comparison, and a persistent job checklist that tracks the whole repair from "vehicle identified" through "test drive completed."

Built as a single self-contained HTML file. State persists in [`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), so you can tap through the flow, close the tab, and come back to your progress.

## What this isn't (yet)

- A working backend. There is no real AI, no real parts lookup, no real shop data. The diagnosis, prices, and shop ratings you see are illustrative.
- A native app. It runs in mobile Safari / Chrome as a web page. Eventually it'll be a [PWA](https://web.dev/progressive-web-apps/) you can add to your home screen.
- Polished. The point is to get the flow right before writing real code.

## Why I'm sharing this

I want feedback on three specific things before I build the actual backend:

1. **Does the flow make sense?** Is there a step that feels redundant or one that's missing?
2. **Is the checklist concept useful?** Right now it tracks the diagnosis steps, then expands into a full repair tracker once you choose DIY or shop. Would you actually use that, or is it noise?
3. **What's the dealbreaker?** Pretend this app existed tomorrow with real data — what's the one thing that would stop you from using it?

Drop an [issue](https://github.com/brendanstiffle-toma/diy-mechanic-genie/issues), comment on the Reddit post, or just tap through and tell me what you think.

## The user flow

Twelve screens, tap-through:

1. Welcome
2. Vehicle ID method (scan VIN / type VIN / pick YMM)
3. VIN camera with live OCR overlay
4. Year / Make / Model / Engine / Trim confirmation
5. Problem capture (photo + text + voice)
6. AI clarifying questions
7. Ranked diagnosis with confidence scores
8. Job overview (difficulty, time, cost range)
9. Parts (with live pricing, vendor links)
10. Tools required (with "you already have this" tracking and AutoZone loan-a-tool flagging)
11. DIY guide (curated YouTube + Reddit threads for *this* exact year/make/model)
12. DIY-vs-shop side-by-side
13. Shops near you (ranked by specialization for this job)

A persistent checklist at the top of every screen tracks the user's overall progress. Phase A (diagnosis) is three items. Phase B expands into the full repair tracker the moment they choose DIY or shop — parts ordered, tools gathered, parts received, repair complete, test drive completed, feedback logged, and so on.

## Planned tech stack (for the real build)

- **Frontend:** [Next.js 14](https://nextjs.org/) + [React](https://react.dev/) + [Tailwind](https://tailwindcss.com/), deployed as a [PWA](https://web.dev/progressive-web-apps/) on [Vercel](https://vercel.com/)
- **Camera + VIN OCR:** [`getUserMedia`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia) + [Tesseract.js](https://tesseract.projectnaptha.com/) in a [Web Worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API), with [Claude Vision](https://docs.claude.com/en/docs/build-with-claude/vision) as the OCR fallback
- **Backend:** [Next.js Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)
- **AI:** [Anthropic Claude](https://www.anthropic.com/claude) ([Sonnet](https://www.anthropic.com/claude/sonnet) for diagnosis, [Haiku](https://www.anthropic.com/claude/haiku) for re-ranking)
- **VIN decode:** [NHTSA vPIC](https://vpic.nhtsa.dot.gov/api/) (free, no key)
- **Parts pricing:** [eBay Browse API](https://developer.ebay.com/api-docs/buy/browse/overview.html) with fitment filter, plus [Amazon PA-API](https://webservices.amazon.com/paapi5/documentation/) for cross-checking
- **Videos:** [YouTube Data API v3](https://developers.google.com/youtube/v3), with Claude Haiku re-ranking the top 10 search results by actual relevance
- **Forums:** [Reddit API](https://www.reddit.com/dev/api/) ([r/MechanicAdvice](https://reddit.com/r/MechanicAdvice), [r/Cartalk](https://reddit.com/r/Cartalk), vehicle-specific subs)
- **Shops:** [Google Places API](https://developers.google.com/maps/documentation/places/web-service/overview) + [RepairPal](https://repairpal.com/) quote ranges
- **Storage:** [Supabase](https://supabase.com/) Postgres

Full architecture and API choices are written up in [diy-mechanic-genie-design.md](./diy-mechanic-genie-design.md).

## Running it locally

```bash
# It's a single file. No build step.
open index.html
```

Or just visit the [live mockup](https://brendanstiffle-toma.github.io/diy-mechanic-genie/) on your phone.

## Roadmap

- [x] Design sketch + clickable mockup
- [ ] **Phase 1:** Next.js skeleton, real VIN decode via NHTSA, vehicle confirm
- [ ] **Phase 2:** Claude diagnosis call with structured JSON output
- [ ] **Phase 3:** eBay Browse API for live parts; YouTube + Reddit surfacing
- [ ] **Phase 4:** Google Places for nearby shops; DIY-vs-shop comparison
- [ ] **Phase 5:** PWA polish, Add to Home Screen, basic analytics

## Contributing

This is a solo prototype right now. The most useful contribution at this stage is feedback — [open an issue](https://github.com/brendanstiffle-toma/diy-mechanic-genie/issues/new) with what works, what doesn't, what's missing. If you're a mechanic or have strong opinions about a specific step, I especially want to hear from you.

## License

[MIT](./LICENSE) — do what you want with it, just keep the copyright notice.
