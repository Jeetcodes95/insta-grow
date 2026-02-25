# Dev Log — Week 01

**Product**: Insta Grow  
**Dates**: Feb 24 – Feb 28, 2026  
**Author**: JeetCodes95  

---

## Monday — Workspace + Auth Foundation

- Multi-tenant workspace schema designed and implemented
- Stripe Customer creation on workspace signup
- OAuth: Instagram (Meta Graph API v18) — install + token storage (AES-256)
- Token refresh scheduler skeleton

**Commits:** 7

---

## Tuesday — Scheduler Architecture

- BullMQ queue system configured (post-scheduler, retry, DLQ)
- ScheduledPost model: content, media, platform, scheduledAt, status
- Cron job: every minute, check for posts due in next 2 minutes
- Instagram publishing worker (Graph API media container → publish flow)
- Posting result recorded + status updated

**Commits:** 8

---

## Wednesday — AI Caption Generator

- OpenAI GPT-4 API integration
- Brand voice prompt template system (user-configurable: professional/casual/bold)
- Caption generation API: `POST /ai/generate-caption` → topic input → caption + CTA variants
- Hashtag generator: trending API scrape + semantic expansion via GPT
- AI credit consumption tracking per workspace

**Design note:** Using GPT-4 Turbo for caption gen, switching to fine-tuned model (v2) once training data available.

**Commits:** 9

---

## Thursday — Comment Classification

- DistilBERT classifier (HuggingFace) deployed in FastAPI service
- Comment ingestion webhook handler (Instagram)
- Classification pipeline: `question | praise | complaint | spam | product_inquiry`
- Auto-reply engine with template responses for praise/spam
- Human review queue for complaints and questions
- Configurable: users can enable/disable auto-reply per account

**Commits:** 10

---

## Friday — Analytics + Rate Limiting

- Analytics snapshot model (daily aggregates per account per platform)
- Instagram Insights ingestion: reach, impressions, follower delta
- Analytics dashboard: follower growth chart, engagement rate card, top posts list
- Rate limiter: Redis sliding window per account per platform
- Automatic pause if platform returns 429 rate-limit header

**Commits:** 7

---

## Week 01 Tags

- Posts scheduled and published: 12 test posts across 2 dev accounts
- AI captions generated: 35 (all human-reviewed and approved)
- Comment classifications tested: 100% accuracy on test set of 50 comments
- All CI checks passing

**Next week:** DM auto-responder, multi-platform (Facebook/LinkedIn), billing logic (plan limits enforcement)
