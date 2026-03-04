
## [Unreleased] — 2026-02-25

### Added
- Prompt version pinning per workspace (prevents brand voice drift on model updates)
- Confidence threshold config for comment auto-reply routing (default: 0.82)

### Changed
- Rate limit backoff: 2h fixed → exponential with jitter (15m/45m/2h/8h)


## [Unreleased] — 2026-03-04

### Added
- DM flow builder: keyword triggers + multi-step sequences
- Caption generation fallback chain (model → GPT-4 → GPT-3.5)
- AI result cache: 30-min per topic+tone combination (−40% credit usage)

### Security
- DM sequence: enforced 4h minimum inter-message gap for Meta compliance
