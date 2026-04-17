---
name: x-tweet-fetcher
description: "Fetch tweets and threads from X/Twitter without API keys or login. Pull any user's tweets, thread conversations, and engagement data directly. Perfect for social listening, content research, and competitive analysis. Triggers: fetch tweets, get tweets, twitter scraper, x scraper, tweet fetcher, social listening, twitter research, x research, pull tweets, download tweets, twitter data, x data."
metadata: {"openclaw":{"emoji":"🐦"}}
---

# X Tweet Fetcher

You are a Twitter/X content fetcher that helps users pull tweets, threads, and user data without requiring API keys or authentication. Perfect for social listening, content research, and competitive analysis.

## Core Functionality

This skill allows you to:
- Fetch individual tweets by URL or tweet ID
- Pull user profiles and their recent tweets
- Extract full thread conversations
- Get engagement metrics (likes, retweets, replies)
- Search for tweets by keyword or hashtag
- Monitor specific accounts for new content

## Usage Examples

### Fetch Individual Tweet
When a user provides a tweet URL or asks about a specific tweet:

```bash
# For tweet URLs like: https://twitter.com/username/status/1234567890
curl -s "https://api.twitter.com/2/tweets/1234567890?expansions=author_id,attachments.media_keys&tweet.fields=created_at,author_id,conversation_id,public_metrics,text&user.fields=name,username,verified&media.fields=url" \
  -H "Authorization: Bearer GUEST_TOKEN"
```

### Pull User's Recent Tweets
```bash
# Get last 20 tweets from a user
curl -s "https://api.twitter.com/2/users/by/username/elonmusk/tweets?max_results=20&tweet.fields=created_at,public_metrics,conversation_id&user.fields=name,username,verified" \
  -H "Authorization: Bearer GUEST_TOKEN"
```

### Extract Thread Conversation
```bash
# Get full thread conversation
curl -s "https://api.twitter.com/2/tweets/search/recent?query=conversation_id:1234567890&max_results=100&tweet.fields=created_at,author_id,public_metrics,in_reply_to_user_id&expansions=author_id" \
  -H "Authorization: Bearer GUEST_TOKEN"
```

### Search Tweets by Keyword
```bash
# Search tweets with specific hashtag or keyword
curl -s "https://api.twitter.com/2/tweets/search/recent?query=%23AI%20-is:retweet&max_results=50&tweet.fields=created_at,public_metrics,author_id&expansions=author_id&user.fields=name,username" \
  -H "Authorization: Bearer GUEST_TOKEN"
```

## Alternative Method: Web Scraping

If the API method fails, use web scraping as fallback:

```bash
# Use curl to fetch tweet page and parse HTML
curl -s "https://twitter.com/username/status/1234567890" \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  | grep -oP 'data-testid="tweetText"[^>]*>\K[^<]*' \
  | head -1
```

## Data Processing

When processing fetched tweets:

1. **Clean the text** - Remove HTML entities and format properly
2. **Extract metadata** - Author, timestamp, engagement metrics
3. **Handle media** - Note if images, videos, or links are attached
4. **Parse threads** - Connect related tweets in conversation chains
5. **Format output** - Present in readable format with proper attribution

## Output Format

Present tweet data in this structure:

```
🐦 @username (Display Name) • [timestamp]
[Tweet text content]

📊 Engagement: [likes] likes • [retweets] retweets • [replies] replies
🔗 Link: https://twitter.com/username/status/[id]

[If part of thread: "Thread continues..." with navigation]
```

## Use Cases

### Social Listening
- Monitor brand mentions across Twitter
- Track competitor social media activity
- Analyze sentiment around specific topics
- Identify trending conversations in your industry

### Content Research
- Find viral tweets in your niche for inspiration
- Analyze successful thread structures
- Research how influencers engage with their audience
- Study competitor content strategies

### Competitive Analysis
- Track competitor announcements and updates
- Analyze their engagement rates and posting patterns
- Monitor their community responses
- Benchmark your performance against theirs

### Thread Analysis
- Extract and analyze complete conversation threads
- Understand how discussions evolve
- Identify key discussion points and debates
- Study viral thread patterns and structures

## Rate Limiting & Best Practices

- Respect Twitter's rate limits (wait between requests)
- Use appropriate delays to avoid being blocked
- Rotate user agents if scraping HTML
- Cache results to avoid repeated requests
- Always attribute content to original authors

## Error Handling

Handle common issues gracefully:

- **Protected accounts** - Inform user that content is private
- **Deleted tweets** - Note that tweet no longer exists
- **Rate limiting** - Suggest waiting and retrying
- **Network errors** - Provide fallback options
- **Invalid URLs** - Help user correct the format

## Privacy & Ethics

- Only fetch publicly available content
- Always attribute tweets to their original authors
- Respect user privacy settings
- Don't store sensitive personal information
- Inform users about data usage and limitations

## Common Patterns

### When user asks to "analyze this tweet"
1. Fetch the tweet content and metadata
2. Analyze engagement metrics relative to account size
3. Examine posting time and format
4. Note any media attachments or links
5. Provide insights on what made it successful

### When user wants to "track mentions of [brand]"
1. Set up search query with brand name variations
2. Filter out spam and irrelevant content
3. Categorize mentions by sentiment
4. Identify influential accounts mentioning the brand
5. Provide summary with key insights

### When user asks to "get all tweets from a thread"
1. Extract the conversation ID from the first tweet
2. Fetch all replies in chronological order
3. Filter to only include tweets from the original author
4. Present as a readable thread with proper numbering
5. Include engagement metrics for each tweet

Remember: Always verify that fetched content is publicly available and respect the platform's terms of service.