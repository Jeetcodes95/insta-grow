# Architecture: Insta Grow System Design

## Social API Integration Layer

```typescript
// Abstract platform adapter pattern
interface SocialPlatformAdapter {
  publishPost(post: ScheduledPost): Promise<PublishResult>;
  getEngagementMetrics(postId: string): Promise<EngagementMetrics>;
  replyToComment(commentId: string, text: string): Promise<void>;
  sendDM(recipientId: string, message: string): Promise<void>;
}

// Platform implementations
class InstagramAdapter implements SocialPlatformAdapter { ... }
class FacebookAdapter implements SocialPlatformAdapter { ... }
class TwitterAdapter implements SocialPlatformAdapter { ... }
class LinkedInAdapter implements SocialPlatformAdapter { ... }
```

## Queue-Based Posting System

```
ScheduledPost created
    │
    ▼
[Scheduler Queue] ← Cron checks every minute
    │
    ▼
[Post Worker] → Platform Adapter → Social API
    │
    ├── Success → Update post status, ingest analytics event
    └── Failure → Retry (3 attempts) → Dead Letter Queue → Alert user
```

### Rate Limiting per Platform

```typescript
const PLATFORM_RATE_LIMITS = {
  instagram: { postsPerHour: 25, commentsPerHour: 60 },
  twitter:   { postsPerHour: 50, commentsPerHour: 100 },
  linkedin:  { postsPerHour: 10, commentsPerHour: 20 },
  facebook:  { postsPerHour: 25, commentsPerHour: 60 },
};

// Redis sliding window rate limiter per account per platform
const canPublish = async (accountId: string, platform: string): Promise<boolean> => {
  const limit = PLATFORM_RATE_LIMITS[platform].postsPerHour;
  const key = `rl:post:${accountId}:${platform}`;
  const count = await redis.incr(key);
  if (count === 1) await redis.expire(key, 3600);
  return count <= limit;
};
```

## AI Content Pipeline

```
User Input (topic / URL / brand brief)
    │
    ▼
[Content Generation Service]
    ├── GPT-4 API call with brand voice prompt
    │   System: "You are writing for {brandName}. Tone: {tone}. Niche: {niche}."
    ├── Hashtag Generator: trending API + semantic expansion
    ├── CTA Optimizer: A/B variant generation (2 options)
    └── Format Adapter: Instagram / Reel / Carousel / LinkedIn Article
    │
    ▼
[Content Queue for Review] → User approval → Scheduling
```

## Comment / DM NLP Engine

```python
# ai-service/comment_classifier/classifier.py
# Multi-label classification: question, praise, complaint, spam, product_inquiry

from transformers import pipeline

classifier = pipeline(
  "text-classification",
  model="distilbert-base-uncased-finetuned-sst-2-english",
  return_all_scores=True
)

REPLY_TEMPLATES = {
  "praise":           "Thank you so much! 🙏 We're glad you love it.",
  "question":         None,              # Escalate to AI response
  "complaint":        None,              # Escalate to human review
  "spam":             "skip",
  "product_inquiry":  None,              # Trigger DM flow
}

def handle_comment(comment_text: str, account_settings: dict) -> dict:
  label = classify(comment_text)
  template = REPLY_TEMPLATES.get(label)
  
  if template == "skip":
    return {"action": "ignore"}
  elif template is None:
    ai_reply = generate_contextual_reply(comment_text, account_settings["tone"])
    return {"action": "reply", "text": ai_reply, "review": True}
  else:
    return {"action": "reply", "text": template, "review": False}
```

## Multi-Tenant Architecture

```typescript
interface Workspace {
  _id: ObjectId;
  ownerId: string;          // Stripe customer ID
  plan: 'solo' | 'agency' | 'scale';
  socialAccounts: SocialAccount[];
  aiCreditsUsed: number;
  aiCreditsLimit: number;
  members: WorkspaceMember[];  // Team access for agency plan
}

interface SocialAccount {
  platform: 'instagram' | 'facebook' | 'twitter' | 'linkedin';
  accountId: string;
  handle: string;
  accessToken: string;       // AES-256 encrypted
  tokenExpiry: Date;
  isActive: boolean;
}
```

## Anti-Spam Compliance

- All automation actions use platform APIs only (no unofficial endpoints or scraping)
- Comment/DM rates enforced well below platform limits (40% of max)
- Human review queue for all AI-generated replies before auto-send (configurable)
- Meta's Messaging Policy compliance: DM responses only to user-initiated conversations
- Rate limit monitoring with automatic pause if anomaly detected
- Comprehensive audit trail: every automated action logged with timestamp and account

## Database Schema

```typescript
// Key collections
Post: { workspaceId, accountId, platform, content, status, scheduledAt, publishedAt, metrics }
Comment: { workspaceId, accountId, platform, commentId, text, classification, replyStatus }
DMConversation: { workspaceId, accountId, platform, participantId, messages[], flowId }
AnalyticsSnapshot: { workspaceId, accountId, platform, date, followers, reach, impressions, engagement }
```

## Scaling Strategy

- Stateless API pods behind load balancer (horizontal scale)
- Queue workers scale independently per platform
- Analytics aggregation batched hourly (not real-time) to reduce API calls
- Redis for job deduplication (prevent double-posting on worker failure)
- MongoDB: per-workspace indexes on all query-heavy collections


## Updated: 2026-02-26

**Comment classifier confidence threshold:** Added configurable confidence threshold per workspace (default: 0.82). Comments scoring below threshold are routed to human review regardless of predicted class. Reduces false auto-replies from 3% to <0.5% of cases in testing.
