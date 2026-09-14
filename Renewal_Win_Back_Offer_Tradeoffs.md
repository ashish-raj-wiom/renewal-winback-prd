# Renewal Win-Back Offer — Tradeoffs Register

Companion to `Renewal_Win_Back_Offer_PRD.md` v1.0, signed off 14 Sep 2026.
Not part of the PRD. This is the record of what was decided and why, so that a question
six months from now — *"why 30 days?"*, *"why no holdout?"* — has an answer without archaeology.

Owner: Ashish Raj · Reviewer: Akash

| # | Decision point | Chosen | Rejected | Why | Date |
|---|---|---|---|---|---|
| 1 | Can the offer's effect be proven? | **No holdout** — ship uncontrolled, add one later | 10% holdout (recommended); 5% holdout | Speed to launch. Accepted with the consequence on the record: ~20% of this cohort returns unaided, so most grants go to customers who were coming back anyway, and that share cannot be measured under this design. Recorded as an Override. | 14 Sep 2026 |
| 2 | What is the reward? | **Set in the offer engine**, not the spec. Launch values 2+2, 7+7, 14+14 | Fixing "match the plan", "half the plan" or "flat bonus" in the PRD | The bonus days are offer data. Changing them should be an offer edit, not a spec change. Values recorded as launch data so engineering has something concrete. | 14 Sep 2026 |
| 3 | What counts as success? | **30 in 100** recharge during the window — about +1,100 customers a month | 25 in 100 (+550); 24 in 100 (+440) | Measured baseline is 19.8%, so this is a ten-point lift. Ambitious for a cohort that has already let 30 days pass. | 14 Sep 2026 |
| 4 | How does the engine know who to target? | **Daily cohort list pushed to the engine** | Live evaluation inside the engine at screen load | Smallest engine change, and whoever owns the cohort definition owns the query. Cost: someone who becomes eligible at 9am waits for the next build. | 14 Sep 2026 |
| 5 | How often do we message an eligible customer? | **Once per entry** | On entry plus a reminder; every daily rebuild (~30 messages) | Safe default for a disengaged audience. Re-messaging people who ignored you is how a lapsed customer becomes a blocked sender. | 14 Sep 2026 |
| 6 | Is "told once" inviolable? | **A V1 default, not a guardrail** | Keeping it as a zero-tolerance invariant | Ops may want to add CleverTap messaging on the same signal later. The spec must not build a hard block that would prevent it. Guardrail G5 was removed and MQ-6 reframed to count announcements by source rather than police them. | 14 Sep 2026 |
| 7 | Two offers that could match the same customer | **Blocked at publish** | Allowing overlap and resolving it per customer with a precedence rule | Makes "one offer per customer" structural rather than a runtime tie-break. | 14 Sep 2026 |
| 8 | Does an open router-pickup disqualify? | **No.** Only a *completed* pickup, or a recharge, takes a customer out | Excluding anyone with a pickup in flight — roughly 600 customers today | The router is still in the house, so a win-back is still possible. The pickup is cancelled when they recharge, which already happens today. | 14 Sep 2026 |
| 9 | Can a customer win back more than once? | **Yes**, whenever they qualify again | One win-back per customer, ever | A customer who lapses again is a genuine R30 case again. Recorded in R4b and tested by AC-SET-5. | 14 Sep 2026 |
| 10 | Does the growth panel need to support this at launch? | **Optional in V1** | Required before launch | Offers can be configured without panel support. The rules hold wherever an offer is defined — panel, API or direct configuration. | 14 Sep 2026 |
| 11 | Are the announcements in V1? | **Yes** — R3 ships in V1; only the copy and the WhatsApp template come later | Deferring the whole announcement capability to a later phase | Considered and reversed: without the announcement the only way a lapsed customer finds the offer is by opening the app unprompted, which is the behaviour the feature exists to cause. | 14 Sep 2026 |
| 12 | A deadline for applying the bonus? | **No recovery window.** The bonus applies with the recharge, or the case is raised to Support/Ops | A 10-minute apply-or-escalate window | The bonus applies in the same operation as the recharge, so a window implied a retry loop nobody is building. Cost: no guaranteed time by which a customer is told. Recorded as an Override. | 14 Sep 2026 |
| 13 | Bounds on the configurable values? | **Defaults only** — ranges read "Not constrained" | Confirming 15–45 for C-01, 45–120 for C-02, daily-only for C-03 | The defaults are the PM's; the ranges were invented and never reviewed. Bounding what growth may set added false precision without adding safety. Recorded as an Override. | 14 Sep 2026 |
| 14 | Test the C-04 "include collected routers" branch? | **Dropped** as not a priority | Keeping an AC for the Yes case | The default (exclude) is covered by AC-SET-2. The Yes branch ships untested by deliberate choice. | 14 Sep 2026 |
| 15 | What do the announcements say? | **Name the best bonus available and route to the recharge screen** | Saying only that an offer is waiting, without a figure | Confirmed at finalise. The words themselves are Comms' to write. | 14 Sep 2026 |
| 16 | Order of the user stories | **Customer value first, adding offers last** | Setup-first ordering | R1 bonus days, R2 sees it, R3 hears about it, then targeting, lifecycle, the regression promise, and setup last. | 14 Sep 2026 |

---

## Two things carried into the build

**The blocker.** A renewal-type offer in `customer-offer-service` currently skips its audience check
entirely and matches every customer with a location on file. It is selectable in the growth panel
today. This must be fixed before any renewal offer is published, including a test one. See §9.

**The measurement dependency.** M1's denominator is "who was in the set", and the set is rebuilt
daily. Unless each day's membership is kept (R4f), that denominator is gone the moment the set
rebuilds, and neither M1 nor any other measurement question can be answered for a past period.
