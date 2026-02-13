# CreatorMetrics

All-in-one analytics dashboard for content creators and influencers.

![Version](https://img.shields.io/badge/version-0.1.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## 🎯 What is CreatorMetrics?

CreatorMetrics helps content creators track their performance across multiple platforms in one unified dashboard. Instead of checking YouTube Studio, Instagram Insights, and Twitter Analytics separately, creators get a single source of truth for:

- Cross-platform follower growth
- Engagement rates and trends
- Audience demographics
- Brand sentiment tracking
- Sponsor-ready reports

## 💰 Pricing

| Plan | Monthly | Annual | Features |
|------|---------|--------|----------|
| Starter | $19 | $190 | 3 platforms, basic analytics |
| Growth | $49 | $490 | 5 platforms, sentiment tracking |
| Pro | $79 | $790 | Unlimited platforms, API access |
| Agency | $199 | $1990 | Multi-account management |

## 🚀 Quick Start (Development)

```bash
# Clone the repository
git clone https://github.com/yourusername/creator-metrics.git
cd creator-metrics

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Add your API keys

# Run database migrations
npm run db:migrate

# Start development server
npm run dev
```

## 🛠️ Tech Stack

- **Frontend:** Next.js 14, Tailwind CSS, shadcn/ui, Recharts
- **Backend:** Node.js, Express, PostgreSQL, Redis
- **Data Collection:** Celery, YouTube API, Instagram API, X API
- **Deployment:** Vercel, Railway

## 📖 Documentation

- [Project Brief](./PROJECT_BRIEF.md) - Full product specification
- [API Documentation](./docs/API.md) - API endpoints and usage
- [Contributing](./CONTRIBUTING.md) - How to contribute

## 📝 Roadmap

- [x] Project planning and design
- [ ] YouTube integration
- [ ] Instagram integration
- [ ] Dashboard UI
- [ ] Stripe payments
- [ ] X/Twitter integration
- [ ] Sentiment analysis
- [ ] TikTok integration
- [ ] Mobile app

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## 📄 License

MIT License - see [LICENSE](./LICENSE) for details.

---

Built with ❤️ for creators
