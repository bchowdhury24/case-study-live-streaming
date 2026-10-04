# Live Streaming Platforms at Scale

**Case study** · Principal/Lead Backend Engineer · 2017 – 2026
`Node.js` `TypeScript` `WebRTC` `Agora SDK` `AWS` `Kubernetes` `Redis` `Socket.IO`

---

## Overview

Over nearly a decade I helped build and launch multiple live streaming
platforms — short-form and interactive video products in the TikTok class —
including one that grew past **$20M in revenue** and **100,000+ monthly active
users**, with **sub-second end-to-end latency** and multi-region failover.

This repository is a public case study. The production code is private;
the architecture, decisions, and lessons are what I'm sharing here.

## The problem

Interactive live streaming is one of the hardest consumer workloads in
backend engineering:

- **Latency is the product.** Above ~2 seconds, the "live" in live streaming
  disappears — gifts, comments, and reactions arrive after the moment has passed.
- **Traffic is spiky and brutal.** A single influencer going live can multiply
  concurrent viewers by 100x in minutes.
- **Cost scales with minutes watched.** Every architectural decision directly
  moves the cloud bill.
- **It must never go down mid-stream.** An outage during a top creator's
  broadcast is a reputational event, not a technical incident.

## Architecture

```
                        ┌─────────────────────────────┐
  Creator ──WebRTC─────▶│  Ingest / SFU layer (Agora) │
                        └──────────────┬──────────────┘
                                       │
                        ┌──────────────▼──────────────┐
                        │  Real-time fanout (Socket.IO)│
                        │  comments · gifts · presence │
                        └──────────────┬──────────────┘
              ┌────────────────────────┼────────────────────────┐
              ▼                        ▼                        ▼
   ┌───────────────────┐  ┌───────────────────┐  ┌───────────────────┐
   │ Presence & chat   │  │ Monetization svc  │  │ Moderation svc    │
   │ (Redis Pub/Sub)   │  │ (payments, gifts) │  │ (AI-assisted)     │
   └───────────────────┘  └───────────────────┘  └───────────────────┘
              │                        │                        │
              └────────────────────────┼────────────────────────┘
                                       ▼
                        ┌─────────────────────────────┐
                        │  MySQL / MongoDB / Redis    │
                        │  CDN + multi-region failover │
                        └─────────────────────────────┘
```

## Key decisions

**1. Buy the video pipeline, own everything around it.**
We used Agora/WebRTC-class infrastructure for transport rather than building
an SFU from scratch — and invested engineering effort where we could
differentiate: the fanout layer, monetization, moderation, and presence.

**2. Redis Pub/Sub for the hot path, databases for the cold path.**
Comments, reactions, and gifts fan out through Redis Pub/Sub at rates a
relational database could never sustain. Everything durable lands in
MySQL/MongoDB asynchronously.

**3. Multi-region from day one, not as a migration.**
Failover was designed into the topology early — moving a platform with live
traffic across regions later is the kind of project that ends careers.

## Results

| Metric | Outcome |
|---|---|
| Scale | 100K+ MAU, [peak concurrent viewers: TBD] |
| Latency | Sub-second end-to-end on interactive streams |
| Revenue | $20M+ generated across platforms I helped build |
| Reliability | Multi-region failover, [uptime: TBD] |

## Lessons

- **Latency budgets are architecture budgets.** Every hop is milliseconds;
  you spend them like money.
- **Spiky traffic demands boring infrastructure.** Autoscaling, queues, and
  backpressure beat cleverness.
- **Moderation can't be an afterthought** at this scale — we integrated
  LLM-assisted moderation into the live path, years before it was standard.

## My role

[CONFIRM: architecture / hands-on split — e.g., "Led backend architecture
and personally built the real-time fanout and presence systems, with a team
of N engineers."]

---

*Code is private (production systems under NDA). Happy to walk through the
architecture in detail — reach me at [email].*
