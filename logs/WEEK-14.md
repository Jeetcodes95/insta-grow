# Dev Log — Week 15

**Product**: Insta Grow  
**Date**: 2026-04-03  
**Author**: JeetCodes95  

---

## This Week

### Completed
- Prompt version pinning per workspace — brand voice configs locked to prompt version at creation
- Rate limit backoff: exponential + jitter (2h fixed → 15min/45min/2h/8h progression)
- AI credit consumption rate: peak usage 3.2 credits/post vs estimated 4.0 — model efficiency better than projected

### In Progress
- DM flow builder UI: keyword triggers, sequence steps, delay config
- Comment classifier confidence threshold: per-workspace setting

### Architecture Note

Implemented caption generation fallback chain:
1. Brand voice model (fine-tuned) → if unavailable → 
2. GPT-4 with brand prompt → if rate limited → 
3. GPT-3.5-turbo with reduced prompt → never fails silently

Users now always get a caption. Quality degrades gracefully under API pressure.

---

## Metrics

| Commits | Workspaces | Posts Scheduled | AI Credits Used |
|---|---|---|---|
| 17 | 4 (dev) | 89 test | 285 |
