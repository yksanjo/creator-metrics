# CreatorMetrics - Technical Specification

## System Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   Web App    │  │  Mobile App  │  │  Chrome Ext  │  │  PDF Reports │ │
│  │   (Next.js)  │  │  (Future)    │  │  (Future)    │  │  (Generated) │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY LAYER                              │
│  ┌─────────────────────────────────────────────────────────────────────┐ │
│  │  Auth (Clerk)  │  Rate Limiting  │  Request Validation  │  Logging │ │
│  └─────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           CORE SERVICES LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Analytics   │  │  Reports     │  │  Billing     │  │  Users       │ │
│  │  Service     │  │  Service     │  │  (Stripe)    │  │  Service     │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           DATA LAYER                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  PostgreSQL  │  │    Redis     │  │ ClickHouse   │  │     S3       │ │
│  │  (Primary)   │  │  (Cache/Queue)│  │ (Analytics)  │  │  (Reports)   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      DATA COLLECTION LAYER                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │   YouTube    │  │  Instagram   │  │   X/Twitter  │  │   TikTok     │ │
│  │  Data Worker │  │  Data Worker │  │  Data Worker │  │  Data Worker │ │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Database Schema

### Core Tables

```sql
-- Users and Authentication
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255),
    avatar_url TEXT,
    plan_type VARCHAR(50) DEFAULT 'starter',
    stripe_customer_id VARCHAR(255),
    stripe_subscription_id VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Platform Connections
CREATE TABLE platform_connections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL, -- 'youtube', 'instagram', 'twitter', 'tiktok'
    platform_user_id VARCHAR(255) NOT NULL,
    access_token TEXT,
    refresh_token TEXT,
    token_expires_at TIMESTAMP,
    metadata JSONB, -- platform-specific data
    is_active BOOLEAN DEFAULT true,
    last_sync_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, platform)
);

-- Daily Metrics (aggregated)
CREATE TABLE daily_metrics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL,
    date DATE NOT NULL,
    followers INTEGER DEFAULT 0,
    following INTEGER DEFAULT 0,
    posts INTEGER DEFAULT 0,
    views INTEGER DEFAULT 0,
    likes INTEGER DEFAULT 0,
    comments INTEGER DEFAULT 0,
    shares INTEGER DEFAULT 0,
    engagement_rate DECIMAL(5,4),
    reach INTEGER DEFAULT 0,
    impressions INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(user_id, platform, date)
);

-- Content Performance
CREATE TABLE content_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL,
    platform_content_id VARCHAR(255) NOT NULL,
    content_type VARCHAR(50), -- 'video', 'image', 'story', 'tweet', etc.
    title TEXT,
    description TEXT,
    thumbnail_url TEXT,
    published_at TIMESTAMP,
    views INTEGER DEFAULT 0,
    likes INTEGER DEFAULT 0,
    comments INTEGER DEFAULT 0,
    shares INTEGER DEFAULT 0,
    engagement_rate DECIMAL(5,4),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(platform, platform_content_id)
);

-- Audience Demographics
CREATE TABLE audience_demographics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL,
    date DATE NOT NULL,
    age_range VARCHAR(20), -- '18-24', '25-34', etc.
    gender VARCHAR(20), -- 'male', 'female', 'unknown'
    country_code VARCHAR(2),
    city VARCHAR(100),
    percentage DECIMAL(5,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Brand Mentions & Sentiment
CREATE TABLE brand_mentions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    platform VARCHAR(50) NOT NULL,
    mention_id VARCHAR(255) NOT NULL,
    content TEXT,
    author_username VARCHAR(255),
    author_followers INTEGER,
    sentiment VARCHAR(20), -- 'positive', 'neutral', 'negative'
    sentiment_score DECIMAL(3,2), -- -1.0 to 1.0
    engagement_count INTEGER DEFAULT 0,
    mentioned_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(platform, mention_id)
);

-- Generated Reports
CREATE TABLE reports (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    report_type VARCHAR(50) NOT NULL, -- 'weekly', 'monthly', 'campaign', 'custom'
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'generating', 'completed', 'failed'
    file_url TEXT,
    file_size INTEGER,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    completed_at TIMESTAMP
);

-- Index for performance
CREATE INDEX idx_daily_metrics_user_platform_date ON daily_metrics(user_id, platform, date);
CREATE INDEX idx_content_user_platform ON content_items(user_id, platform);
CREATE INDEX idx_content_published ON content_items(published_at DESC);
CREATE INDEX idx_mentions_user_sentiment ON brand_mentions(user_id, sentiment);
```

---

## API Endpoints

### Authentication
```
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh
POST   /api/v1/auth/forgot-password
POST   /api/v1/auth/reset-password
```

### Platform Connections
```
GET    /api/v1/platforms              # List connected platforms
POST   /api/v1/platforms/:platform/connect
DELETE /api/v1/platforms/:platform/disconnect
POST   /api/v1/platforms/:platform/refresh
GET    /api/v1/platforms/:platform/status
```

### Analytics
```
GET    /api/v1/analytics/overview     # Dashboard overview
GET    /api/v1/analytics/growth       # Growth trends
GET    /api/v1/analytics/engagement   # Engagement metrics
GET    /api/v1/analytics/demographics # Audience demographics
GET    /api/v1/analytics/content      # Content performance
GET    /api/v1/analytics/compare      # Cross-platform comparison
```

Query Parameters for Analytics:
- `platforms`: Comma-separated list (youtube,instagram,twitter)
- `start_date`: YYYY-MM-DD
- `end_date`: YYYY-MM-DD
- `granularity`: day, week, month

### Sentiment & Mentions
```
GET    /api/v1/sentiment/overview     # Sentiment summary
GET    /api/v1/sentiment/timeline     # Sentiment over time
GET    /api/v1/mentions               # Brand mentions list
GET    /api/v1/mentions/:id           # Single mention detail
```

### Reports
```
GET    /api/v1/reports                # List reports
POST   /api/v1/reports                # Generate new report
GET    /api/v1/reports/:id            # Get report details
GET    /api/v1/reports/:id/download   # Download PDF
DELETE /api/v1/reports/:id            # Delete report
```

### User & Billing
```
GET    /api/v1/user/profile
PUT    /api/v1/user/profile
GET    /api/v1/user/subscription
POST   /api/v1/user/subscription/upgrade
POST   /api/v1/user/subscription/cancel
GET    /api/v1/user/usage             # API usage limits
```

---

## Data Sync Architecture

### Sync Strategy by Platform

| Platform | Sync Frequency | Method | Rate Limits |
|----------|----------------|--------|-------------|
| YouTube | Every 6 hours | API + Webhooks | 10,000 units/day |
| Instagram | Every 6 hours | Graph API | 200 calls/hour |
| X/Twitter | Every 4 hours | API v2 | 300 requests/15min |
| TikTok | Daily | Research API | Limited access |

### Background Job Queue

```python
# Celery tasks structure

@celery.task
def sync_user_platforms(user_id):
    """Sync all platforms for a user"""
    connections = get_active_connections(user_id)
    for conn in connections:
        sync_platform.delay(user_id, conn.platform)

@celery.task(bind=True, max_retries=3)
def sync_platform(self, user_id, platform):
    """Sync single platform with retry logic"""
    try:
        connector = get_connector(platform)
        data = connector.fetch_metrics(user_id)
        store_metrics(user_id, platform, data)
    except RateLimitError as exc:
        self.retry(countdown=3600)  # Retry in 1 hour
    except APIError as exc:
        logger.error(f"API error syncing {platform}: {exc}")
        raise

@celery.task
def generate_report(report_id):
    """Generate PDF report"""
    report = get_report(report_id)
    data = aggregate_report_data(report)
    pdf = create_pdf(data)
    upload_to_s3(report_id, pdf)
    notify_user(report.user_id, "Report ready")

# Scheduled tasks
celery.conf.beat_schedule = {
    'sync-all-users': {
        'task': 'tasks.sync_all_users',
        'schedule': crontab(hour='*/6'),  # Every 6 hours
    },
    'cleanup-old-data': {
        'task': 'tasks.cleanup_old_metrics',
        'schedule': crontab(hour=3, minute=0),  # Daily at 3am
    },
}
```

---

## Caching Strategy

### Redis Cache Keys

```
# User session
user:session:{session_id} -> JWT payload (TTL: 24h)

# Platform data (short-term)
platform:{user_id}:{platform}:metrics -> JSON (TTL: 1h)
platform:{user_id}:{platform}:followers -> Integer (TTL: 15min)

# Aggregated analytics (medium-term)
analytics:{user_id}:{hash_of_params} -> JSON (TTL: 6h)

# Reports (long-term)
reports:{user_id}:list -> JSON (TTL: 1h)
reports:{report_id}:data -> JSON (TTL: 24h)

# Rate limiting
ratelimit:{user_id}:{endpoint} -> Counter (TTL: varies)
```

### Cache Invalidation

```javascript
// When platform sync completes
redis.del(`platform:${user_id}:${platform}:metrics`);
redis.del(`analytics:${user_id}:*`); // Wildcard delete pattern

// When user upgrades plan
redis.del(`user:${user_id}:subscription`);
```

---

## Security Considerations

### Data Protection
- All tokens encrypted at rest (AES-256)
- TLS 1.3 for all API communications
- Row-level security for multi-tenant data
- Regular token rotation for platform APIs

### API Security
```javascript
// Rate limiting middleware
const rateLimit = {
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: (req) => {
    const plan = req.user.plan;
    return plan === 'pro' ? 1000 : plan === 'growth' ? 500 : 100;
  }
};

// Request validation
const validateAnalyticsQuery = [
  body('platforms').isArray().isIn(['youtube', 'instagram', 'twitter']),
  body('start_date').isISO8601(),
  body('end_date').isISO8601(),
  validate
];
```

### OAuth Flow Security
- PKCE for mobile/single-page apps
- State parameter validation
- Short-lived auth codes (10 minutes)
- Secure cookie settings (HttpOnly, SameSite)

---

## Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Dashboard load | <2s | Time to interactive |
| API response | <500ms | P95 latency |
| Report generation | <30s | PDF creation time |
| Data freshness | <6 hours | Last sync timestamp |
| Uptime | 99.9% | Monthly availability |

---

## Deployment

### Docker Compose (Development)

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/creatormetrics
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

  db:
    image: postgres:16
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=creatormetrics

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

  worker:
    build: .
    command: celery -A tasks worker --loglevel=info
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/creatormetrics
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis

volumes:
  postgres_data:
  redis_data:
```

### Production (Railway/Vercel)

```bash
# Environment variables
DATABASE_URL=
REDIS_URL=
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
CLERK_SECRET_KEY=
YOUTUBE_API_KEY=
INSTAGRAM_APP_ID=
INSTAGRAM_APP_SECRET=
TWITTER_API_KEY=
TWITTER_API_SECRET=
OPENAI_API_KEY=
```

---

## Monitoring & Observability

### Key Metrics to Track
- API response times (P50, P95, P99)
- Platform sync success/failure rates
- Database query performance
- Cache hit rates
- User engagement (DAU/MAU)
- Error rates by endpoint

### Logging
```javascript
// Structured logging
logger.info({
  event: 'platform_sync_completed',
  userId: user.id,
  platform: 'youtube',
  duration: 2345, // ms
  recordsProcessed: 156,
  timestamp: new Date().toISOString()
});
```

### Alerting Rules
- Sync failure rate > 5% for any platform
- API error rate > 1%
- Database connection pool exhausted
- Redis memory usage > 80%
- Failed payment rate > 10%

---

## Future Enhancements

### Phase 2
- Real-time WebSocket updates
- AI-powered content recommendations
- Competitor tracking
- Influencer marketplace integration

### Phase 3
- Mobile apps (iOS/Android)
- Team collaboration features
- White-label options
- Advanced API for enterprises
