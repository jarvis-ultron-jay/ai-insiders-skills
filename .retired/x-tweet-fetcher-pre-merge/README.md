# X Tweet Fetcher — Social Media Intelligence Without API Keys

Fetch tweets, threads, and user data from X/Twitter without authentication. Perfect for social listening, content research, and competitive analysis.

## Features

- **Individual Tweet Fetching** — Pull any public tweet by URL or ID
- **User Timeline Access** — Get recent tweets from any public account
- **Thread Extraction** — Extract complete conversation threads
- **Engagement Metrics** — Retrieve likes, retweets, and reply counts
- **Keyword Search** — Find tweets by hashtags or keywords
- **Account Monitoring** — Track specific accounts for new content
- **No API Keys Required** — Works without Twitter API access

## Install

```bash
npx clawhub install x-tweet-fetcher
```

## Usage Examples

Just tell your AI assistant what you need:

| Task | Example prompts |
|------|----------------|
| Individual Tweet | "Fetch this tweet: [URL]", "Get the tweet with ID 1234567890" |
| User's Tweets | "Get Elon Musk's last 10 tweets", "Show me recent posts from @username" |
| Thread Analysis | "Extract this full thread", "Get all replies in this conversation" |
| Keyword Search | "Find tweets about #AI", "Search for mentions of 'ChatGPT'" |
| Social Listening | "Monitor mentions of our brand", "Track competitor activity" |
| Content Research | "Find viral tweets in our niche", "Analyze successful thread patterns" |

## Use Cases

### Social Listening
- Monitor brand mentions across Twitter
- Track competitor social media activity  
- Analyze sentiment around specific topics
- Identify trending conversations in your industry

### Content Research
- Find viral tweets in your niche for inspiration
- Analyze successful thread structures
- Research how influencers engage with audiences
- Study competitor content strategies

### Competitive Analysis
- Track competitor announcements and updates
- Analyze engagement rates and posting patterns
- Monitor community responses
- Benchmark performance against competitors

### Thread Analysis
- Extract and analyze complete conversation threads
- Understand how discussions evolve
- Identify key discussion points and debates
- Study viral thread patterns and structures

## Output Format

The skill presents tweet data in a clean, readable format:

```
🐦 @username (Display Name) • 2h ago
This is the tweet content with proper formatting
and line breaks preserved.

📊 Engagement: 42 likes • 12 retweets • 3 replies
🔗 Link: https://twitter.com/username/status/1234567890
```

For threads, it connects related tweets and shows the conversation flow.

## Privacy & Best Practices

- Only accesses publicly available content
- Respects rate limits to avoid blocking
- Always attributes content to original authors
- Handles protected accounts and deleted tweets gracefully
- Follows ethical scraping practices

## Technical Approach

The skill uses a combination of:
- Guest token API access (when available)
- Intelligent HTML parsing as fallback
- Smart rate limiting and request rotation
- Robust error handling for various edge cases

No authentication required — works out of the box for any public Twitter content.

## Based On

This skill is inspired by the open-source [x-tweet-fetcher](https://github.com/ythx-101/x-tweet-fetcher) project by ythx-101 (771 GitHub stars), which pioneered authentication-free Twitter data access.

---

**Note**: This tool is for accessing publicly available information only. Always respect platform terms of service and user privacy.