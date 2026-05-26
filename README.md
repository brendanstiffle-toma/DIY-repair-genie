DIY Mechanic Genie
A mobile-first web app concept that turns "something's wrong with my car" into a confident decision: fix it yourself with the right parts and a trustworthy video, or hand it to a shop with a fair quote in your pocket.

Try the clickable mockup →

Status: early prototype, mocked data, looking for feedback before building the real thing.


What this is
A clickable HTML mockup of every screen in the user flow — VIN scan, vehicle confirmation, problem capture, AI diagnosis, parts list, tools, curated YouTube + Reddit guides, DIY-vs-shop comparison, and a persistent job checklist that tracks the whole repair from "vehicle identified" through "test drive completed."

Built as a single self-contained HTML file. State persists in localStorage, so you can tap through the flow, close the tab, and come back to your progress.
What this isn't (yet)
A working backend. There is no real AI, no real parts lookup, no real shop data. The diagnosis, prices, and shop ratings you see are illustrative.
A native app. It runs in mobile Safari / Chrome as a web page. Eventually it'll be a PWA you can add to your home screen.
Polished. The point is to get the flow right before writing real code.
Why I'm sharing this
I want feedback on three specific things before I build the actual backend:

Does the flow make sense? Is there a step that feels redundant or one that's missing?
Is the checklist concept useful? Right now it tracks the diagnosis steps, then expands into a full repair tracker once you choose DIY or shop. Would you actually use that, or is it noise?
What's the dealbreaker? Pretend this app existed tomorrow with real data — what's the one thing that would stop you from using it?

Drop an issue, comment on the Reddit post, or just tap through and tell me what you think.
The user flow
Twelve screens, tap-through:

Welcome
Vehicle ID method (scan VIN / type VIN / pick YMM)
VIN camera with live OCR overlay
Year / Make / Model / Engine / Trim confirmation
Problem capture (photo + text + voice)
AI clarifying questions
Ranked diagnosis with confidence scores
Job overview (difficulty, time, cost range)
Parts (with live pricing, vendor links)
Tools required (with "you already have this" tracking and AutoZone loan-a-tool flagging)
DIY guide (curated YouTube + Reddit threads for this exact year/make/model)
DIY-vs-shop side-by-side
Shops near you (ranked by specialization for this job)

A persistent checklist at the top of every screen tracks the user's overall progress. Phase A (diagnosis) is three items. Phase B expands into the full repair tracker the moment they choose DIY or shop — parts ordered, tools gathered, parts received, repair complete, test drive completed, feedback logged, and so on.
Planned tech stack (for the real build)
Frontend: Next.js 14 + React + Tailwind, deployed as a PWA on Vercel
Camera + VIN OCR: getUserMedia + Tesseract.js in a Web Worker, with Claude Vision as the OCR fallback
Backend: Next.js Route Handlers
AI: Anthropic Claude (Sonnet for diagnosis, Haiku for re-ranking)
VIN decode: NHTSA vPIC (free)
Parts pricing: eBay Browse API with fitment filter, plus Amazon PA-API for cross-checking
Videos: YouTube Data API v3, with Claude Haiku re-ranking the top 10 search results by actual relevance
Forums: Reddit API (r/MechanicAdvice, r/Cartalk, vehicle-specific subs)
Shops: Google Places API + RepairPal quote ranges
Storage: Supabase Postgres

Full architecture and API choices are written up in diy-mechanic-genie-design.md.
Running it locally
# It's a single file. No build step.
open index.html

Or just visit the Pages URL on your phone.
Roadmap
Design sketch + clickable mockup
Phase 1: Next.js skeleton, real VIN decode via NHTSA, vehicle confirm
Phase 2: Claude diagnosis call with structured JSON output
Phase 3: eBay Browse API for live parts; YouTube + Reddit surfacing
Phase 4: Google Places for nearby shops; DIY-vs-shop comparison
Phase 5: PWA polish, Add to Home Screen, basic analytics
Contributing
This is a solo prototype right now. The most useful contribution at this stage is feedback — open an issue with what works, what doesn't, what's missing. If you're a mechanic or have strong opinions about a specific step, I especially want to hear from you.
License
MIT — see LICENSE.

