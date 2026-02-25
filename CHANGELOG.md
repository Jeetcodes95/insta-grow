
## [Unreleased] — 2026-02-25

### Added
- Prompt version pinning per workspace (prevents brand voice drift on model updates)
- Confidence threshold config for comment auto-reply routing (default: 0.82)

### Changed
- Rate limit backoff: 2h fixed → exponential with jitter (15m/45m/2h/8h)
