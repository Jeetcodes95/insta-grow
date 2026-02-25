# Security & Compliance: Insta Grow

## Authentication & Token Security

- Social platform OAuth tokens encrypted with AES-256-GCM (key in AWS KMS)
- Tokens decrypted only at execution time — never stored in plaintext or logged
- Token refresh jobs run 24 hours before expiry to prevent posting failures
- Workspace JWT: 15-minute access + 7-day refresh (same pattern as Super CRM)

## Social API Compliance

### Meta Graph API (Instagram / Facebook)
- Only official Graph API endpoints — no private/unofficial APIs
- DM automation restricted to user-initiated conversations only (Meta policy compliance)
- Rate limits respected at 40% of platform maximum (buffer for burst)
- Data retention: user PII not stored beyond token storage; post content ephemeral

### Twitter/X API v2
- OAuth 2.0 PKCE flow
- Rate limit headers honored; 429 responses trigger automatic backoff with jitter
- Automated posting clearly within Basic tier limits

### LinkedIn API
- Share API + UGC Post API used exclusively
- Community Management API for comment handling on organization pages

## Anti-Spam Architecture

```typescript
// All automation actions subject to these enforcements:
const SAFETY_LIMITS = {
  instagram: {
    followsPerHour: 20,         // Platform cap: 60 — we use 33%
    commentsPerHour: 30,        // Platform cap: 60
    dmPerHour: 10,              // Platform cap: 20
  },
  twitter: {
    postsPerHour: 15,           // Well under Basic tier limit
    repliesPerHour: 30,
  }
};

// Anomaly detection: if 429 count spikes, pause account for 2 hours
async function handleRateLimitError(accountId: string, platform: string) {
  await redis.set(`pause:${accountId}:${platform}`, '1', 'EX', 7200);
  await notifyWorkspaceOwner(accountId, `${platform} automation paused: rate limit protection active`);
}
```

## Human Review Queue

All AI-generated content that requires sensitive handling goes through review before execution:
- Comment replies classified as `complaint` or `question` → human review mandatory
- DM sequences > 3 messages deep → human approval required
- Posts for regulated industries (finance, health) → mandatory review flag per account setting

## Data Privacy

- No customer/follower PII stored — only anonymized engagement signals
- GDPR data export: workspace owner can download all stored data
- GDPR deletion: account deletion triggers cascade purge within 24 hours
- Analytics data anonymized before ML model training
