# CreatorMetrics API Documentation

## Base URL
```
Production: https://api.creatormetrics.com/v1
Development: http://localhost:3000/api/v1
```

## Authentication

All API requests require an `Authorization` header with a Bearer token:

```
Authorization: Bearer <your_api_token>
```

## Rate Limiting

| Plan | Requests per 15 min |
|------|---------------------|
| Starter | 100 |
| Growth | 500 |
| Pro | 1000 |

Rate limit headers included in all responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1640995200
```

---

## Endpoints

### Analytics

#### Get Dashboard Overview
```http
GET /analytics/overview
```

Returns aggregated metrics for the dashboard.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| platforms | string | No | Comma-separated: `youtube,instagram,twitter` |
| period | string | No | `7d`, `30d`, `90d`, `1y` (default: 30d) |

**Response:**
```json
{
  "data": {
    "total_followers": 245231,
    "follower_growth": 12340,
    "follower_growth_pct": 5.29,
    "avg_engagement_rate": 4.2,
    "engagement_change": 0.5,
    "total_reach": 1200000,
    "reach_change": 8.3,
    "platforms": [
      {
        "platform": "youtube",
        "followers": 125000,
        "growth": 5200,
        "engagement_rate": 5.1
      },
      {
        "platform": "instagram",
        "followers": 89231,
        "growth": 5140,
        "engagement_rate": 3.8
      }
    ],
    "recent_content": [
      {
        "id": "vid_123",
        "platform": "youtube",
        "title": "My Latest Video",
        "views": 45000,
        "engagement_rate": 6.2,
        "published_at": "2024-01-15T10:00:00Z"
      }
    ]
  }
}
```

#### Get Growth Trends
```http
GET /analytics/growth
```

Returns historical follower and engagement data.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| platforms | string | Yes | Comma-separated platform list |
| start_date | string | Yes | ISO 8601 date |
| end_date | string | Yes | ISO 8601 date |
| granularity | string | No | `day`, `week`, `month` |

**Response:**
```json
{
  "data": {
    "platforms": ["youtube", "instagram"],
    "granularity": "day",
    "metrics": [
      {
        "date": "2024-01-01",
        "youtube": {
          "followers": 120000,
          "following": 0,
          "posts": 45,
          "engagement_rate": 4.8
        },
        "instagram": {
          "followers": 85000,
          "following": 450,
          "posts": 320,
          "engagement_rate": 3.5
        }
      }
    ]
  }
}
```

#### Get Audience Demographics
```http
GET /analytics/demographics
```

Returns audience demographic breakdown.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| platform | string | No | Specific platform or `all` |

**Response:**
```json
{
  "data": {
    "age_distribution": [
      { "range": "18-24", "percentage": 35.2 },
      { "range": "25-34", "percentage": 42.1 },
      { "range": "35-44", "percentage": 18.5 },
      { "range": "45+", "percentage": 4.2 }
    ],
    "gender_distribution": [
      { "gender": "male", "percentage": 48.5 },
      { "gender": "female", "percentage": 49.2 },
      { "gender": "unknown", "percentage": 2.3 }
    ],
    "top_countries": [
      { "code": "US", "name": "United States", "percentage": 45.2 },
      { "code": "GB", "name": "United Kingdom", "percentage": 12.3 },
      { "code": "CA", "name": "Canada", "percentage": 8.7 }
    ],
    "top_cities": [
      { "name": "New York", "country": "US", "percentage": 8.2 },
      { "name": "Los Angeles", "country": "US", "percentage": 6.5 }
    ]
  }
}
```

---

### Platforms

#### List Connected Platforms
```http
GET /platforms
```

**Response:**
```json
{
  "data": [
    {
      "platform": "youtube",
      "status": "connected",
      "connected_at": "2024-01-01T00:00:00Z",
      "username": "MyChannel",
      "followers": 125000,
      "last_sync": "2024-01-15T12:00:00Z"
    },
    {
      "platform": "instagram",
      "status": "connected",
      "connected_at": "2024-01-02T00:00:00Z",
      "username": "myhandle",
      "followers": 89231,
      "last_sync": "2024-01-15T12:00:00Z"
    },
    {
      "platform": "twitter",
      "status": "disconnected",
      "error": "token_expired"
    }
  ]
}
```

#### Connect Platform
```http
POST /platforms/:platform/connect
```

Initiates OAuth flow for platform connection.

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| platform | string | Yes | `youtube`, `instagram`, `twitter`, `tiktok` |

**Request Body:**
```json
{
  "redirect_uri": "https://app.creatormetrics.com/callback"
}
```

**Response:**
```json
{
  "data": {
    "auth_url": "https://accounts.google.com/o/oauth2/auth?...",
    "state": "random_state_string"
  }
}
```

#### Disconnect Platform
```http
DELETE /platforms/:platform/disconnect
```

Removes platform connection and deletes associated data.

---

### Sentiment

#### Get Sentiment Overview
```http
GET /sentiment/overview
```

Returns sentiment analysis summary.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| period | string | No | `7d`, `30d`, `90d` |

**Response:**
```json
{
  "data": {
    "total_mentions": 1250,
    "sentiment_breakdown": {
      "positive": 67.5,
      "neutral": 25.3,
      "negative": 7.2
    },
    "sentiment_score": 0.72,
    "trend": "improving",
    "top_positive_topics": ["content quality", "helpful advice", "entertaining"],
    "top_negative_topics": ["upload frequency", "audio quality"],
    "platform_breakdown": {
      "twitter": { "mentions": 450, "sentiment": 0.68 },
      "instagram": { "mentions": 520, "sentiment": 0.75 },
      "youtube": { "mentions": 280, "sentiment": 0.71 }
    }
  }
}
```

#### Get Mentions
```http
GET /mentions
```

Returns list of brand mentions.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sentiment | string | No | `positive`, `neutral`, `negative` |
| platform | string | No | Filter by platform |
| limit | number | No | Default: 20, Max: 100 |
| cursor | string | No | Pagination cursor |

**Response:**
```json
{
  "data": [
    {
      "id": "mention_123",
      "platform": "twitter",
      "content": "Just discovered @creator - amazing content! 🔥",
      "author": {
        "username": "fan123",
        "display_name": "John Doe",
        "followers": 1250,
        "avatar_url": "https://..."
      },
      "sentiment": "positive",
      "sentiment_score": 0.85,
      "engagement": {
        "likes": 45,
        "replies": 3,
        "shares": 12
      },
      "mentioned_at": "2024-01-15T10:30:00Z",
      "url": "https://twitter.com/..."
    }
  ],
  "pagination": {
    "next_cursor": "eyJpZCI6MTIzfQ==",
    "has_more": true
  }
}
```

---

### Reports

#### List Reports
```http
GET /reports
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| limit | number | No | Default: 10 |
| cursor | string | No | Pagination |

**Response:**
```json
{
  "data": [
    {
      "id": "rep_123",
      "type": "monthly",
      "status": "completed",
      "start_date": "2024-01-01",
      "end_date": "2024-01-31",
      "platforms": ["youtube", "instagram"],
      "file_url": "https://cdn.creatormetrics.com/reports/...",
      "file_size": 2450000,
      "created_at": "2024-02-01T00:00:00Z",
      "completed_at": "2024-02-01T00:00:30Z"
    }
  ]
}
```

#### Generate Report
```http
POST /reports
```

**Request Body:**
```json
{
  "type": "monthly",
  "start_date": "2024-01-01",
  "end_date": "2024-01-31",
  "platforms": ["youtube", "instagram", "twitter"],
  "include_sentiment": true,
  "include_demographics": true
}
```

**Response:**
```json
{
  "data": {
    "id": "rep_124",
    "status": "generating",
    "estimated_completion": "2024-02-01T00:01:00Z",
    "webhook_url": "https://api.creatormetrics.com/webhooks/report-complete"
  }
}
```

#### Download Report
```http
GET /reports/:id/download
```

Returns a signed URL for PDF download (expires in 1 hour).

**Response:**
```json
{
  "data": {
    "download_url": "https://cdn.creatormetrics.com/reports/...?signed=true",
    "expires_at": "2024-02-01T01:00:00Z"
  }
}
```

---

## Error Handling

All errors follow this format:

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "API rate limit exceeded. Please try again in 15 minutes.",
    "details": {
      "retry_after": 900
    }
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Invalid or missing API token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `PLATFORM_ERROR` | 502 | Connected platform API error |
| `INTERNAL_ERROR` | 500 | Server error |

---

## Webhooks

Subscribe to real-time events:

```http
POST /webhooks/subscribe
{
  "url": "https://your-app.com/webhooks",
  "events": ["report.completed", "platform.disconnected"]
}
```

### Event Types

| Event | Description |
|-------|-------------|
| `report.completed` | PDF report generation finished |
| `platform.connected` | New platform connected |
| `platform.disconnected` | Platform connection lost |
| `platform.sync_failed` | Data sync error |
| `subscription.updated` | Plan changed |

### Webhook Payload

```json
{
  "event": "report.completed",
  "timestamp": "2024-01-15T12:00:00Z",
  "data": {
    "report_id": "rep_123",
    "user_id": "usr_456",
    "download_url": "https://cdn.creatormetrics.com/reports/..."
  }
}
```

---

## SDKs

### JavaScript

```bash
npm install @creatormetrics/sdk
```

```javascript
import { CreatorMetrics } from '@creatormetrics/sdk';

const client = new CreatorMetrics({
  apiKey: 'your_api_key'
});

// Get dashboard overview
const overview = await client.analytics.overview({
  platforms: ['youtube', 'instagram'],
  period: '30d'
});

// Generate report
const report = await client.reports.generate({
  type: 'monthly',
  start_date: '2024-01-01',
  end_date: '2024-01-31'
});
```

### Python

```bash
pip install creatormetrics
```

```python
from creatormetrics import Client

client = Client(api_key="your_api_key")

# Get growth data
growth = client.analytics.growth(
    platforms=["youtube", "instagram"],
    start_date="2024-01-01",
    end_date="2024-01-31"
)

# Get mentions
mentions = client.mentions.list(sentiment="positive", limit=50)
```

---

## Changelog

### v1.0.0 (2024-02-01)
- Initial API release
- YouTube, Instagram, X/Twitter support
- Basic analytics and reporting
- Sentiment analysis

### v1.1.0 (Upcoming)
- TikTok integration
- Real-time webhooks
- Advanced demographic analysis
