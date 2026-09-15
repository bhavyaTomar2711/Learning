# 09b · The Domain Auction Lifecycle

**Internal. Do not share outside Namekart.** This document describes how our business actually makes money, including our registrar rights and our bidding strategy.

Adapted for the curriculum from the canonical domain-knowledge reference owned by Yash. Read it on Day 0, before you trace the golden path, and again before your first acquisition-related ticket.

Everything in the auction service and the acquisition side of the dashboard exists because of what is described here. The code changes fast; this reality changes slowly. When the two disagree, the code should adapt to the reality.

Two markers are used throughout:

- **[Namekart]** marks a fact specific to us, such as our registrar rights.
- **[Variable]** marks a mechanic that can change by platform, TLD, or season, and must never be hardcoded.

---

## 0. The short version

- A domain is not a static asset. An expiring domain travels a **lifecycle**: auction, closeout, pending delete, drop, sometimes a registry dropzone, then normal registration. It can be **acquired at several points along that path, on several competing platforms**.
- The same domain can appear on **multiple platforms at once**. Sometimes that is one auction mirrored across sites. Sometimes those are independent competing routes, and only the one that captures the domain charges us.
- Acquisition cost differs wildly by stage and platform, so **a single "max price" is the wrong model.** We reason in **stage-specific ceilings**: "in a live auction, go up to X; if it falls to closeout, only take it at Y".
- The job is to **capture the domains we want at the lowest cost with acceptable probability**, across parallel routes, informed by buyer demand, at scale, increasingly with AI.

---

## 1. Owned versus non-owned domains, and "LTD"

| Term | Meaning |
|---|---|
| **Owned / portfolio** | Domains we have acquired and hold for resale, development, or parking. The portfolio side of the dashboard. |
| **Non-owned / acquisition pipeline** | Domains we are *trying* to acquire: in auction, closeout, drop, or dropzone. The entire Acquisition Center. |
| **LTD** | Originally "Limited Time Domain": a non-owned domain available only for a limited window. In our product it now means **sales-assisted acquisition during a live auction**: Sales works the domain, gathering real buyer demand, *before* the bidding closes, so the acquisition decision is informed by actual market interest. An LTD domain is still a non-owned pipeline row, not a portfolio asset. |

**Why it matters:** buyer demand discovered while a domain is still in auction can justify a higher, better-informed ceiling. LTD is the bridge between the sales pipeline and the acquisition pipeline.

---

## 2. How a domain becomes available

An expiring domain generally moves through these stages. Not every domain hits every stage, and **the route a domain will follow is largely predictable up front** (section 2.2).

```
Registered
  (owner lets it expire; grace and redemption periods per registry policy)
  -> Registrar expired auction       competitive ascending bidding
  -> Closeout / Dutch auction        descending "last chance" price on the registrar (GoDaddy, Dynadot)
       or Namecheap extended auction  Namecheap extends about a day at a lower floor instead
  -> Pending delete                  registrar releases it toward deletion
       + drop-catch backorders        buyers pre-order with catchers (DropCatch, GName, NameJet, SnapNames)
                                      to grab it the instant it deletes; several backorders for one
                                      domain means the catcher runs a private auction among them
  -> Drop (deletion)                 the domain actually deletes
       + registry dropzone            on Identity Digital TLDs, a multi-day descending-price window
                                      before normal availability
  -> Normal availability             anyone can register at the standard TLD price
```

**Why registrars and registries do this.** A registrar does not want to delete potentially valuable inventory; deletion is lost revenue. The expired auction, the closeout, and the extension all exist **to get someone to buy before deletion**, extracting whatever the market will bear at each step. Registries run dropzones for the same reason. These are monetisation tactics, not neutral infrastructure. Our edge is knowing the mechanics well enough to buy at the cheapest viable stage.

**Key intuition.** Earlier stages mean more demand and a higher price. Later stages mean the domain went uncontested and is available cheaper. A domain nobody bid on in auction is, by revealed preference, less contested, and can often be captured at the lowest price the lifecycle allows. **Smart acquisition is buying at the right stage for the right price.**

### 2.1 Drop-catching and backorders

Drop-catching is a first-class route, not a footnote.

- A domain in **pending delete** will delete after a set number of days and then be available at normal registration price.
- **Drop-catch services** accept **backorders**: you pre-order, and their infrastructure races to re-register the domain the moment it deletes.
- If several parties backordered the same domain, the catcher that wins runs a **private auction among the backorderers**.
- **No capture is guaranteed.** A different catcher may win the drop. That is why drop-catch usually runs in parallel with other routes (section 4).

### 2.2 Predicting a domain's route

Each party's mechanics are known and fairly stable, so **at ingestion we can predict the stages a domain will probably pass through**, and set ceilings for each from day one.

| Factor | What it determines |
|---|---|
| **Current registrar** | Which auction and post-auction mechanic applies. GoDaddy and Dynadot: expired auction plus a closeout ladder. Namecheap: auction plus an extension of about a day, no multi-day closeout. Each has its own marketplace, fees, and timing. |
| **Registry** (the TLD's backend operator) | Whether a registry dropzone exists at all. Identity Digital runs one across the many TLDs it operates. Verisign, for `.com` and `.net`, does not, so a `.com` never has a dropzone stage. |
| **TLD** | The normal registration price, whether the TLD is premium, which registry operates it, and any TLD-specific layering such as Namecheap's `.ai` play (section 3.7). |
| **Shared-inventory policy** | Whether the auction is mirrored across platforms or exclusive to one. |
| **[Namekart] Our EPP rights on the TLD** | Whether direct registry capture (section 3.6) is available to us, a route others may not have. |
| **Demand** (observed, not known up front) | How far down the lifecycle a domain travels. Wanted names sell early in auction; unwanted ones fall through to closeout, drop, and dropzone floor. |

**Design consequence:** route prediction must be a **rule- and data-driven function of registrar, registry, TLD, our rights, and shared-inventory policy**. Never hardcode it to one TLD or platform. All of those sets grow and change.

---

## 3. Platforms and their mechanics

### 3.1 Shared inventory [Variable]

Big registrars sometimes share expired-auction inventory: the same auction appears on several marketplaces at once and **bids are synced**. It is one auction with several user interfaces. You can bid through any of them, usually preferring the original registrar, but it is **one acquisition event**. Not all inventory is shared.

### 3.2 Expired auctions

Standard ascending auctions on the registrar's marketplace. If a domain gets bids, it goes to the highest bidder at close.

### 3.3 Closeout / Dutch auctions: GoDaddy, Dynadot [Variable prices]

When an expired auction ends with **no bids**, the registrar runs a **closeout**: a descending-price auction where the price drops on a fixed schedule over several days until someone buys at the current price, first come first served, or the domain exits to pending delete.

- An illustrative ladder: day 1 $50, day 2 $40, day 3 $30, day 4 $11, day 5 $5.
- **Strategy:** if we want a domain that received no auction bids, we generally want it at the **lowest** closeout price, not the first day's, because "no bids" signals low contention. Hence a separate, lower ceiling for the closeout stage (section 5).

### 3.4 Namecheap extended auction [Variable]

Instead of a closeout ladder, Namecheap **extends the auction by about a day at a reduced minimum price**. It is still an ascending auction, not a buy-now, just with a lower floor.

### 3.5 Pending delete and drop-catching [Variable]

If still uncaptured, the domain enters pending delete, is deleted after a set number of days, and becomes available at normal registration price. Drop-catch services take backorders, race for the registration, and run private auctions when several parties backordered. Drop-catch is a parallel route with its own cost profile, a backorder fee plus a possible auction, and **no guarantee of capture**.

### 3.6 Identity Digital registry dropzone [Variable; Namekart has partial registry access]

Some **registries**, not registrars, run their own descending-price windows. Identity Digital's dropzone:

- A **Dutch auction of about 14 days**, open for **one hour each day**, with the price falling daily. After that, the domain is available for normal registration.
- **The dropzone spans the many TLDs Identity Digital operates**, such as `.io`, `.org`, and many more. **It is not specific to `.ai`, and must never be hardcoded to `.ai`.** `.ai` is prominent for us because it is valuable, because Namecheap layers an extra auction on it (section 3.7), and because it is one of the TLDs where we hold registrar rights today. The multi-path TLD configuration reflects current footprint, not a limit.
- **[Namekart]** We are a registrar on Identity Digital with **EPP access to registry operations** on a subset of its TLDs, including `.ai` and `.me`. That lets us attempt **direct EPP capture** during the dropzone window. We aim to expand that set; every TLD added unlocks the cheapest capture route for that TLD.

### 3.7 Namecheap's `.ai` dropzone auction [Variable; time-sensitive]

This is **a monetisation play by Namecheap**, not a neutral marketplace feature. Namecheap layers its own auction on top of Identity Digital's `.ai` dropzone, timed so it can **buy low from the dropzone and resell to its own bidders**. It earns largely from bidders who do not realise the same domain is often available cheaper directly through the dropzone. Knowing this lets us avoid overpaying and pursue the direct route where we can.

- Namecheap starts a **5-day auction with a $400 minimum**, beginning on the day the domain sits in the $400 dropzone price window.
- It runs until the domain reaches the **$25 window on day 5**. With no bid by then, Namecheap **extends the auction at a $10 minimum**, matching the $10 dropzone window.
- **Namecheap's `.ai` auctions end at 6:30 PM IST**, and the **dropzone daily window runs 6:45 to 7:45 PM IST** [Variable]. So once someone bids, Namecheap captures the domain in the next dropzone window: at a profit if the bid came at the $400-minimum stage or above, at break-even if it came only at the $10 stage.

**Why this matters to us:** for `.ai` we can pursue a domain **both** through Namecheap's auction, with our "NC-DZ" ceiling, **and** through our own EPP capture in the dropzone window, with our "DZ" ceiling.

---

## 4. Parallel acquisition: two different cases

One domain can be pursued through several routes at once. **There are two kinds of "parallel", and they must be modelled differently.**

### 4.1 Type A: synced shared inventory, which is really one acquisition

Several platforms show the **same auction** with synced bids.

- **Model it as** one acquisition event. Bid through any surfacing platform, preferring the original registrar.
- **Charged by** the one auction. There is no double-charge risk.

### 4.2 Type B: competing independent routes

Genuinely different mediums each try to capture the domain **independently, and compete**. For example: Namecheap's `.ai` auction plus our own EPP capture in the dropzone window; or a registrar auction plus a drop-catch backorder on the eventual deletion.

- **Only the medium that captures the domain charges us.**
- **No medium guarantees capture.** That is why we run several. If we capture through EPP, that is often cheapest. If Namecheap captures it for us, that is also fine when the domain mattered enough.

### 4.3 The cost versus probability trade-off

This is the heart of acquisition strategy.

- Ideally we buy through the medium that charges least.
- Cheaper mediums often have lower capture probability.
- So for a highly valued domain, with strong leads or a likely buyer, we may **approve a higher ceiling on a higher-probability route while also trying the cheaper route**. Capturing the domain is what matters; the route is secondary.
- Therefore **willingness to pay cannot be one number.** It needs per-stage, per-route ceilings.

---

## 5. Pricing: stage-specific APRs, recos, and order splits

### 5.1 Why one ceiling is wrong

A fair acquisition price depends on the stage and route of capture, because each carries different information about contention and different fees. For one domain:

- In the **GoDaddy live auction**, the ceiling is `apr = 25`: we compete up to $25 if others bid.
- If it falls to **closeout**, nobody bid, so the ceiling is `apr_buy = 5`. We do not want it at the first closeout price of $11; we want it at the $5 floor.

That is the difference between paying $25, $11, or $5 for the *same* domain, compounded across thousands of domains.

### 5.2 The set of stage and route ceilings

Currently these are columns on the domains table, and the set is meant to grow.

| Ceiling | Applies to | Meaning |
|---|---|---|
| **APR** (`apr`) | Normal expired auctions on Dynadot and GoDaddy; Namecheap phases 1 and 2 for multi-path TLDs | Maximum bid in the live auction |
| **APR Buy** (`apr_buy`) | Dynadot and GoDaddy closeout stage only | Maximum at closeout, deliberately lower than `apr` |
| **APR NC-DZ** (`apr_nc_dz`) | Namecheap phases 3 and 4, the `.ai` dropzone re-list | Maximum for the Namecheap dropzone route |
| **APR DZ** (`apr_dz`) | Identity Digital dropzone, direct EPP capture | Registry fee cap for our own capture. **NULL means do not pursue. 0 means capture only in the free final-day window. Greater than 0 means the maximum bid.** |

**NULL and 0 mean different things.** Not pursuing a route is different from pursuing it only at the free floor. Any future per-route ceiling must keep that three-way distinction: skip, floor, or capped.

### 5.3 Slots and timing [Variable]

Namecheap routes carry a **preferred slot**: which window or phase to target, because the same domain moves through Namecheap's phases at different times of day, aligned to the dropzone window. Timing is strategic and specific to TLD and registry.

### 5.4 Recos versus APRs

- A **reco** is a *proposed* ceiling for a stage or route. Several acquisition employees can each propose reco sets, and AI agents propose them too, ideally from day one, as soon as the domain's route can be predicted.
- An **APR** is a *finalised* reco: the committed ceiling the bidding system actually respects.
- **[Namekart]** Today only admins finalise APRs. As APR agents prove reliable, the intent is to let them set APRs within guardrails such as confidence thresholds, caps, and human review of borderline cases.

### 5.5 Two ways to turn recos into bids

1. **Stage-specific ceilings** (primary): set each stage's and route's ceiling independently. This best expresses "cheap at closeout, higher in auction".
2. **Order split** (complementary): a single maximum decided by a human or AI is **split across** the stage ceilings or platforms. Useful when the decision starts from "we will spend up to $X to capture this".

Both produce the per-route ceilings the bid dispatcher consumes.

### 5.6 Mapping the route at ingestion

When a domain first appears in an expired auction on any platform, predict its probable route from the factors in section 2.2, and **set the reco and APR set for those stages up front**, instead of reacting stage by stage.

---

## 6. How domains enter the pipeline

- **Feed ingestion.** Drop lists and expired-auction inventory arrive from registrars and marketplaces such as GoDaddy, Dynadot, Namecheap, DropCatch, and GName. **Estibot is a valuation and enrichment source, not the discovery feed**: it adds appraisal data to a domain; it does not decide what enters the pipeline.
- **User-uploaded lists.** Analysts can upload arbitrary domain lists for opportunities the feed does not surface. They enter the same pipeline and are shortlistable.
- **AI is one input, not the source of truth.** AI shortlisting and scoring is one signal among several. Humans can shortlist and propose recos independently, and AI never gates human action.

---

## 7. Live auction dynamics and the dynamic APR

Once an APR is set, the domain may still be in a live auction whose dynamics change: the price crosses thresholds, new bidders join, time runs out. The direction we are heading:

- **Watch live auction data** after the APR is set.
- **Adjust the APR dynamically** based on auction dynamics (price, bidders, time left), buyer signals (a strong buyer found through LTD justifies paying more), and intrinsic quality.
- This is the natural home of a future **dynamic-APR AI agent**, reacting to live auction events and buyer demand, per route, within guardrails.

Signals such an agent can use: live auction state (current price, bid count, bidders, time left, status); buyer demand (leads, strong-lead count, price signals from buyers); intrinsic valuation (appraisal sources); and the domain's current lifecycle stage.

---

## 8. Constants versus variables

**Treat as relatively constant, and model as stable first-class concepts:**

- The lifecycle shape: auction, closeout, pending delete, drop, registry dropzone, normal registration.
- Descending-price mechanics at closeout and dropzone stages.
- Stage- and route-specific ceilings.
- The two kinds of parallelism: synced shared inventory, and competing routes where the winner charges.
- The reco-to-APR authority flow, with humans and agents proposing.
- Buyer demand informing price.

**Treat as variable, and drive from configuration or data, never hardcode:**

- Which platforms, registrars, and catchers exist.
- Which TLDs we have registrar and EPP rights on.
- Price ladders, and which TLDs are multi-path.
- Timing windows: the dropzone hour, Namecheap's close time, the 14-day cadence.
- Minimum prices, fees, and slot definitions.

**Rule of thumb:** a new platform to observe or a new order route should be addable through configuration or data, with no schema change. A genuinely new *type* of ceiling is a deliberate, reviewed schema extension. The long-term shape is a normalised `(domain, route, ceiling, timing)` model rather than a fixed set of `apr_*` columns, so a fifth competing route is additive rather than a rewrite.

---

## 9. What "good" looks like

1. **Buy at the right stage.** Prefer the cheapest stage with acceptable capture probability for a domain of that value. Unbid domains are taken at the closeout or dropzone floor, not the top.
2. **Run competing routes for valuable domains.** Accept a higher ceiling on a high-probability route while a cheaper route runs in parallel. Only the winner charges.
3. **Let demand raise the ceiling.** Wire buyer signals into the pricing decision.
4. **Front-load recos.** Map the probable route at ingestion and set stage ceilings from day one.
5. **Scale with AI, keep humans in the loop.** AI recos and agents for throughput; human override and admin finalisation for borderline and valuable cases.
6. **Expand registrar and EPP footprint.** More TLDs with direct registry access means more cheapest-route captures.
7. **Avoid stale decisions.** Auctions move; re-evaluate ceilings on meaningful events.

---

## 10. Where the product is heading

- A **per-route ceiling model**, normalising the `apr_*` columns into `(domain, route, ceiling, timing)`.
- A **parallel-route order model** recording which competing routes are active for a domain and reconciling "the winner charges".
- A **dynamic-APR agent** driven by live auction events and buyer signals, which needs event wiring (price crossing, new bidder, time remaining, close) and signal fusion.
- **Completing the LTD workflow**, so buyer demand gathered during a live auction actually feeds the APR.
- **Route prediction at ingestion**, pre-populating reco slots.
- **A multi-author reco model**: parallel reco sets from several analysts and agents, versus one current value with history.
- **Registry operation visibility**: surfacing EPP results and errors separately from marketplace bid results.

---

## 11. Where you will meet this in code

- **Auction service:** registrar integrations, auction monitoring, bid dispatch, and dropzone capture through the EPP service. Look for how stage, platform, and ceiling decide whether and how much to bid.
- **Dashboard backend:** the acquisition center, shortlisting, reco and APR fields on the domains table, auction state tracking, and lifecycle status.
- **The golden path (L2)** is exactly the handoff from "a human or AI shortlists a domain with ceilings" to "the auction service acts on them".

When a field, enum, or branch in that code looks arbitrary, find the section of this document that explains it before you change it.

---

## 12. Glossary

| Term | Definition |
|---|---|
| **APR** | The finalised ceiling the bidding system will pay for a domain on a given stage or route. |
| **Reco** | A proposed ceiling for a stage or route, from an analyst or an AI agent, before it is finalised. |
| **LTD** | A non-owned domain in the pipeline; in our product, specifically the sales-assisted acquisition during a live auction. |
| **Expired auction** | An ascending auction on a registrar's marketplace for a lapsed domain. |
| **Closeout** | A descending-price last-chance sale after an expired auction got no bids. |
| **Dutch auction** | An auction whose price falls over time until someone buys. |
| **Pending delete** | The stage before a lapsed domain is deleted and becomes generally available. |
| **Drop-catch** | Re-registering a domain the instant it deletes, done by specialist catchers. |
| **Backorder** | A pre-order to capture a domain on drop; several backorders lead to a private auction. |
| **Shared inventory** | One auction shown on several platforms with synced bids. |
| **Dropzone** | A registry-run descending-price window, such as Identity Digital's roughly 14-day window with a daily hour. |
| **EPP** | Extensible Provisioning Protocol, used between registrars and registries. We can capture directly on TLDs where we are a registrar. |
| **NC-DZ** | Namecheap's auction layered on Identity Digital's `.ai` dropzone. |
| **Slot / phase** | Which timed window of a multi-phase route a bid targets. |
| **Parallel acquisition** | Pursuing one domain through several routes: synced (one event) or competing (the winner charges). |
| **Order split** | Distributing one maximum budget across stage-specific ceilings or platforms. |

---

## Self-check

1. A `.com` domain gets no bids in its GoDaddy auction. List every remaining stage it could pass through, and the cheapest point at which we could take it.
2. Why is a `.io` domain's route different from a `.com` domain's, even on the same registrar?
3. A domain is shown on two marketplaces. How do you decide whether that is Type A or Type B parallelism, and why does the answer change what the system must record?
4. What is the difference between `apr_dz = NULL` and `apr_dz = 0`, and what bug would you create by treating them the same?
5. Where in our code would hardcoding `.ai` be a mistake, and how should it be expressed instead?
6. Which parts of this document would change if a new registry launched its own dropzone next month, and which should not?
