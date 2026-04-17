# X Toolkit

Everything your agent needs to work with X/Twitter data — from public tweet scraping to processing your own bookmark library. One skill, two workflows.

## What It Does

**Public tweet fetching (no login required):**
- Fetch any user's recent tweets
- Pull full thread conversations from a tweet URL
- Extract engagement metrics (likes, retweets, replies)
- Track growth patterns and content performance over time

**Your bookmarks (requires OAuth):**
- Pull your entire bookmark library
- Categorise saved tweets by topic automatically
- Extract key insights from bookmarked threads
- Convert saves into action items, tasks, or content ideas
- Weekly bookmark review and cleanup

## Requirements

- OpenClaw installed and running
- No API keys required for public tweet fetching
- For bookmark processing: X/Twitter account + `bird` CLI or X API v2 OAuth (~5 min setup)

## Example Prompts

**Public fetching:**
- "Fetch the last 20 tweets from @elonmusk"
- "Pull the full thread from this tweet URL"
- "What's @naval posted about AI in the last 30 days?"

**Bookmarks:**
- "Process my X bookmarks from this week"
- "What actionable items are in my recent bookmarks?"
- "Turn my last 50 bookmarks into a content calendar"

The skill handles mode detection automatically — no manual configuration needed.
