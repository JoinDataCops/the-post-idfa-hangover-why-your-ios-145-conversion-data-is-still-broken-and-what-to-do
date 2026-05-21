# The Post-IDFA Hangover: Why Your iOS 14.5+ Conversion Data Is Still Broken (And What to Do)

**April 26, 2021 was the day a quarter of the internet went dark for Facebook advertisers.** That is the date iOS 14.5 shipped App Tracking Transparency. Five years later, your CPAs still have not recovered. You deployed the [Conversions API](/conversion-api) like every guide told you to. You still feel the hangover.

I have rebuilt Meta tracking for dozens of accounts since that update. Here is the honest read: **CAPI did not fix the problem. It papered over the part you can see and left the part you cannot.**

Every article on this topic stops at "set up CAPI, recover your conversions." That is a measurement post. This is not a measurement post. This is a post about **what those recovered conversions actually do to Meta's algorithm once they arrive**, because most of them are not real conversions at all. They are guesses Meta dressed up to look like data.

**The lie is that iOS 14.5 broke your tracking and CAPI fixed it.** The truth is iOS 14.5 broke your data quality, and CAPI faithfully delivers low-quality data into a system that learns from it. DataCops exists because the fix is architectural: clean, first-party signals filtered before they leave your infrastructure, not modeled signals stitched back together after the fact.

## Quick stuff people keep asking

**Why is my Meta ads [attribution](/resources/cross-channel-attribution-setup-bridging-the-silos) still broken in 2026?** Because CAPI restored the pipe, not the signal. Roughly 75% of iOS users opt out of ATT. Meta cannot see them individually, so it models them. Modeled conversions are statistical estimates, not events. Your attribution is "working" and "wrong" at the same time.

**Does [Meta Conversions API](/meta-conversion-api) fully replace the Facebook pixel?** No. Run both, deduplicated by event ID. CAPI is server-side so ITP and ad blockers cannot strip it the way they strip the browser pixel. But CAPI still depends on what your server actually knows about the visitor. If the session was anonymous and consent-gated, CAPI has thin data to send.

**What is Aggregated Event Measurement and do I need it?** AEM is Meta's client-side workaround for opted-out users. You rank up to 8 conversion events per verified domain, and Meta reports them in aggregate with deliberate noise and delay. If you advertise to iOS users, you are already using it whether you configured it well or not. Most accounts have not touched the priority order in years.

**How much data did iOS 14.5 actually cost Facebook advertisers?** Meta itself flagged roughly $10B in 2022 revenue impact. For individual advertisers the visible loss was 15-25% of reported conversions overnight, with worse gaps in iOS-heavy verticals.

**What percentage of iOS users opt out of IDFA tracking?** Opt-in sits around 20-25% depending on the vertical and the prompt. So 75-80% of your iOS audience is invisible to deterministic, user-level tracking.

**Is my reported [ROAS](/resources/facebook-roas-improvement-guide-from-black-box-to-profit-engine) lower because of iOS privacy changes?** Partly. Some of the ROAS drop is real lost attribution. Some is the attribution window shrinking from 28-day to 7-day click as the default, which moves conversions out of the reporting frame entirely. And some is the algorithm genuinely underperforming because it is learning from bad signals. Three causes, one symptom.

**What is the difference between SKAdNetwork and CAPI?** SKAdNetwork is Apple's privacy framework: it reports install and post-install events with a coarse conversion value, delayed and aggregated, no user-level detail. CAPI is your server sending events directly to Meta. SKAN is what Apple lets you see. CAPI is what you choose to send. They answer different questions and neither is complete.

**Can server-side tracking fully recover lost iOS conversion data?** No. It recovers signal that ad blockers and ITP would have stripped client-side. It cannot recover consent you never got or identity the user never shared. Anyone promising full recovery is selling you modeled data and calling it found data.

## The hangover is a feedback loop, not a tracking gap

Here is the part nobody indexes.

Meta's bidding system is a learning machine. It does not just report conversions. It studies them, builds a profile of who converts, and goes hunting for more people who look like that profile. The quality of the people it finds is entirely a function of the quality of the conversions you feed it.

Post-IDFA, a large share of the conversions Meta works with are modeled. For opted-out iOS users it cannot observe the real event, so it estimates: this cohort, this campaign, this much spend, statistically this many conversions probably happened. Then it attributes those modeled conversions to profiles it guessed at. Then it optimizes toward those guessed profiles.

Read that chain again. IDFA removed identity. Modeling filled the hole with estimates. The algorithm trained on the estimates. It now spends your budget chasing an audience that was never confirmed to exist.

> That is the hangover. Not "I lost 20% of my conversions." It is "the 80% I still see are teaching the algorithm a slightly wrong lesson, every single day, and the error compounds."

Now stack the second contaminant on top. The events your server does capture cleanly are not all human. Across the open web, 24-31% of what looks like converting traffic is automated. Bots fill forms. Bots complete checkouts on stolen cards. Bots click ads and land on your page and trip your conversion event. CAPI does not know the difference. It hashes the email, packages the event, and ships it to Meta as a genuine conversion.

I watched this play out at a company called PillarlabAI. They ran a honeypot on their signup flow to find out how dirty their funnel really was. Three thousand signups came in. Seventy-seven percent were fraudulent. And here is the detail that should make you put your coffee down: 650 of those accounts traced back to a single device fingerprint. One machine. Six hundred and fifty "conversions."

If those signups fire a conversion event, CAPI sends 650 of them to Meta. Meta does not see one bot. It sees 650 happy customers and asks itself what they have in common. It builds a lookalike. It spends your money finding more machines exactly like that one. Garbage in, garbage optimized, garbage out.

That is why your [CPA](/resources/cost-per-acquisition-cpa-optimization-lower-costs-higher-profits) never came back. You fixed the pipe. You never cleaned the water.

## What "fixed" actually requires

The competing guides treat this as a config problem. Add CAPI. Add event ID deduplication. Hash your PII with SHA-256. Set your AEM priority. All correct, all necessary, and all of it operates on data that is already contaminated before it reaches the configuration.

The order is the whole point. Data quality first, then implementation. If the pre-conversion funnel is full of bots and the human sessions are getting blocked, perfect tracking just reports the garbage faithfully and at higher fidelity. You have made the wrong number more precise.

Fixing the foundation means three things, all architectural:

You collect first-party, on your own infrastructure, on your own subdomain, so ITP and ad blockers cannot quietly delete a third of your real human signal before you ever see it. Resilient collection, far harder to strip than a third-party browser pixel.

You filter bots at ingestion, before any event becomes a "conversion." A 361.8B-plus IP reputation database separates residential humans from datacenter, VPN, proxy, and Tor traffic at the moment of collection. The 650-accounts-on-one-fingerprint case gets surfaced before it ever becomes a CAPI payload. Meta never gets the chance to learn from it.

You separate your data into two tiers at the source. Anonymous session analytics flow unconditionally and legally. Identifiable, consented events flow with consent attached. You stop blending the two and stop sending Meta a smear of confirmed humans, modeled guesses, and bots labeled identically.

That is DataCops. First-party architecture, [bot filtering](/fraud-traffic-validation) at ingestion, CAPI to Meta, Google, TikTok, and LinkedIn from one clean pipeline. Two honest caveats so the rest lands straight: [SOC 2 Type II](/enterprise) is in progress, so a regulated buyer may want to wait for it, and DataCops is a newer brand than the legacy attribution vendors. Worth knowing before you commit.

## Decision guide

You deployed CAPI and your CPA is flat. Audit data quality before you touch a single campaign setting. Tracking is not your problem.

You run iOS-heavy paid social. Treat every reported conversion as a mix of confirmed, modeled, and bot. Stop reading the dashboard as literal truth.

You have never revisited your AEM priority order. Do it this week. Put the event closest to revenue at the top.

You see your reported ROAS sliding and you are scaling spend anyway. Stop scaling. Scaling amplifies whatever the algorithm learned, and right now it learned from contaminated signal.

You are choosing between three attribution dashboards. None of them fixes this. They re-model the same dirty data three different ways. Fix collection first.

## You are not measuring wrong. You are training wrong.

The mistake I see, on nearly every account, is treating the post-IDFA hangover as a reporting inconvenience. Numbers look low, the thinking goes, but spend keeps working, so leave it.

Spend is not working. It is being optimized against a blend of real conversions, statistical fiction, and bot activity, and the algorithm cannot tell which is which because you never gave it the chance to. Every day that loop runs, it tunes a little more precisely toward the wrong people.

CAPI did not end the hangover. It hid the symptom and let the cause keep training your most expensive automated system against your own interests.

So here is the question. Of the conversions Meta reported to you last month, how many can you prove were a real human who actually bought? If you cannot put a number on that, you are not running a campaign. You are running an experiment, and the algorithm is the only one being taught anything.

---

Research by [DataCops](https://www.joindatacops.com) — first-party tracking, consent infrastructure, fraud prevention, and server-side CAPI for Meta, Google, TikTok, and LinkedIn.
