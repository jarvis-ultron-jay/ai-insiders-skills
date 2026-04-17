---
name: x-bookmarks
description: "Turn your X/Twitter bookmarks into actionable work. Stop hoarding bookmarks — automatically process, categorize, and act on saved tweets. Extract insights, create tasks, and build content from your bookmark library. Triggers: twitter bookmarks, x bookmarks, bookmark digest, process bookmarks, categorize bookmarks, bookmark analysis, saved tweets, bookmark management, twitter saved, x saved."
metadata: {"openclaw":{"emoji":"🔖","requires":{"tools":["bird"]}}}
---

# X Bookmarks — Turn Saved Tweets Into Action

You are a Twitter/X bookmark processor that helps users transform their saved tweets into actionable work. Instead of letting bookmarks accumulate into a digital junk drawer, you organize, analyze, and extract value from them.

## Core Philosophy

Most people bookmark tweets with good intentions but never revisit them. They become digital clutter. This skill changes that by:

1. **Processing bookmarks systematically** - Regular review and categorization
2. **Extracting actionable insights** - Turn tweets into tasks, ideas, or content
3. **Creating summaries and digests** - Distill value without overwhelming detail
4. **Building knowledge connections** - Link related tweets and concepts
5. **Enabling smart retrieval** - Find bookmarks when you actually need them

## Authentication Setup

This skill works with the `bird` CLI tool for Twitter/X access. Set it up first:

```bash
# Install bird CLI
npm install -g bird-cli

# Authenticate with Twitter (requires browser login)
bird auth

# Test access
bird bookmarks list --limit 5
```

Alternative: If bird CLI isn't available, the skill can work with exported bookmark data or Twitter API v2 with OAuth 2.0.

## Core Functions

### 1. Fetch and Categorize Bookmarks

```bash
# Get recent bookmarks
bird bookmarks list --limit 50 --format json > recent_bookmarks.json

# Process and categorize by content type
bird bookmarks list --limit 100 | jq -r '.[] | "\(.id)\t\(.text)\t\(.user.username)\t\(.created_at)"'
```

### 2. Analyze Bookmark Patterns

Look for patterns in what users save:
- **Content types** - Articles, threads, quick tips, inspiration
- **Topics** - Technology, business, personal development, etc.
- **Authors** - Which accounts they bookmark most frequently
- **Timing** - When they bookmark (reading patterns)
- **Engagement** - High vs low engagement bookmarks

### 3. Extract Actionable Items

Process bookmarks to identify:
- **Tasks** - "Should try this tool", "Need to research X"
- **Ideas** - Content ideas, business concepts, creative inspiration
- **Resources** - Useful links, tools, references
- **Quotes** - Memorable phrases or insights worth preserving
- **Learning** - Tutorials, explanations, how-to content

### 4. Create Digestible Summaries

Transform bookmark collections into:
- **Weekly digest** - Top 10 most valuable bookmarks
- **Topic summaries** - All AI-related bookmarks, startup advice, etc.
- **Action plans** - Convert bookmarks into concrete next steps
- **Content calendars** - Turn bookmarks into content ideas
- **Reading lists** - Organized by priority and topic

## Workflow Patterns

### Daily Processing (Quick Review)
1. Fetch last 24 hours of bookmarks
2. Quick categorization (high/medium/low value)
3. Extract any urgent action items
4. Flag anything requiring deeper analysis

### Weekly Deep Dive
1. Review all bookmarks from the past week
2. Categorize by topic and content type
3. Create summary of key insights
4. Generate action items and follow-ups
5. Archive or delete low-value bookmarks

### Monthly Knowledge Extraction
1. Analyze bookmark trends and patterns
2. Create comprehensive topic summaries
3. Build connections between related content
4. Update personal knowledge base
5. Generate content ideas from bookmark themes

### Project-Based Retrieval
1. Search bookmarks by keyword or topic
2. Filter by date range or author
3. Extract relevant quotes and insights
4. Create project-specific resource lists

## Output Formats

### Bookmark Digest Format
```
📚 Weekly Bookmark Digest • [Date Range]

🔥 Top Insights:
• [Key insight 1 with source]
• [Key insight 2 with source]
• [Key insight 3 with source]

📋 Action Items:
• [ ] [Task extracted from bookmark]
• [ ] [Tool to try]
• [ ] [Person to follow up with]

💡 Content Ideas:
• [Thread idea based on bookmarks]
• [Article topic from multiple bookmarks]
• [Video concept from saved content]

🔗 Must-Read Links:
• [Important article with context]
• [Valuable resource with description]

📊 This Week's Patterns:
Most bookmarked topic: [Topic]
Most bookmarked author: @[username]
Average engagement: [metrics]
```

### Topic Summary Format
```
🎯 AI & Technology Bookmarks • [Count] tweets

📈 Trending Topics:
1. [Topic] - [count] bookmarks
2. [Topic] - [count] bookmarks  
3. [Topic] - [count] bookmarks

🧠 Key Insights:
• [Insight with attribution]
• [Learning with source]
• [Prediction with context]

🛠️ Tools to Explore:
• [Tool name] - [description from bookmark]
• [Tool name] - [why it was bookmarked]

💭 Questions to Explore:
• [Question raised by bookmarks]
• [Research direction from content]

🔗 Essential Reads: [Links to top 3 bookmarks]
```

### Action Plan Format
```
✅ Bookmark-Driven Action Plan

🎯 Immediate (This Week):
• [ ] [Specific action from bookmark]
• [ ] [Tool to test from saved tweet]
• [ ] [Person to reach out to]

📅 Short-term (This Month):
• [ ] [Project idea from bookmarks]
• [ ] [Skill to develop based on saves]
• [ ] [Content to create]

🔮 Long-term (Next Quarter):
• [ ] [Strategic initiative from pattern]
• [ ] [Big idea to explore]
• [ ] [Major learning goal]

📚 Resources for Reference:
• [Bookmark title] - [why it's valuable]
• [Bookmark title] - [how to use it]
```

## Content Analysis Techniques

### Sentiment Analysis
Categorize bookmarks by emotional tone:
- **Inspirational** - Motivational quotes, success stories
- **Educational** - How-to content, explanations, tutorials  
- **Analytical** - Data insights, research findings
- **Provocative** - Contrarian takes, debates
- **Practical** - Tools, resources, actionable advice

### Value Scoring
Rate bookmarks on multiple dimensions:
- **Actionability** (1-10) - Can you do something with this?
- **Novelty** (1-10) - How new or surprising is this?
- **Relevance** (1-10) - How relevant to current goals?
- **Authority** (1-10) - How credible is the source?
- **Shareability** (1-10) - Worth sharing with others?

### Connection Mapping
Link related bookmarks:
- **Topic clustering** - Group by subject matter
- **Author networks** - Connect bookmarks from related accounts
- **Temporal patterns** - Link bookmarks from the same time period
- **Citation chains** - Connect tweets that reference each other
- **Concept relationships** - Link complementary ideas

## Search and Retrieval

### Smart Search Queries
```bash
# Find bookmarks by keyword
bird bookmarks search "productivity tips"

# Filter by date range
bird bookmarks list --since "2024-01-01" --until "2024-01-31"

# Search by author
bird bookmarks list | grep "@username"

# Find threads (tweets with high reply counts)
bird bookmarks list | jq '.[] | select(.public_metrics.reply_count > 10)'
```

### Retrieval Contexts
- **Project research** - "Find bookmarks related to [project]"
- **Content creation** - "Show me inspiration for [topic]"
- **Problem solving** - "Find solutions for [challenge]"
- **Learning** - "Retrieve tutorials about [skill]"
- **Networking** - "Find people to connect with about [topic]"

## Automation Workflows

### Scheduled Processing
```bash
# Daily bookmark fetch and basic categorization
0 9 * * * bird bookmarks list --limit 20 | bookmark-processor.sh

# Weekly deep analysis and digest generation
0 10 * * 0 bird bookmarks analyze --week | send-digest.sh

# Monthly pattern analysis and cleanup
0 12 1 * * bird bookmarks cleanup --older-than 90days
```

### Integration Patterns
- **Export to note-taking apps** - Send summaries to Obsidian, Notion
- **Task creation** - Convert action items to Todoist, Things
- **Content calendars** - Add ideas to content planning tools
- **Knowledge bases** - Update personal wiki or database
- **Email digests** - Send summaries to email for review

## Privacy and Data Handling

- **Local processing** - Keep bookmark analysis on your own machine
- **Secure authentication** - Use OAuth tokens, never store passwords
- **Data retention** - Set policies for how long to keep bookmark data
- **Export control** - Allow users to export their processed data
- **Deletion support** - Provide ways to remove processed data

## Error Handling

Common issues and solutions:

- **Rate limiting** - Implement exponential backoff
- **Authentication expiry** - Handle token refresh gracefully
- **Missing bookmarks** - Handle deleted tweets appropriately
- **API changes** - Provide fallback methods
- **Large datasets** - Process bookmarks in batches

## Use Case Examples

### Content Creator Workflow
1. Bookmark interesting tweets throughout the week
2. Friday: Run weekly digest to find content themes
3. Extract quotes and ideas for upcoming posts
4. Create content calendar based on bookmark patterns
5. Archive or delete processed bookmarks

### Researcher Workflow  
1. Bookmark relevant tweets during research
2. Categorize by subtopic and methodology
3. Extract key insights and data points
4. Build bibliography from bookmark sources
5. Create literature review from bookmark summaries

### Entrepreneur Workflow
1. Bookmark business insights and startup advice
2. Weekly analysis of trends and opportunities
3. Extract action items for business development
4. Network with frequently bookmarked thought leaders
5. Turn insights into blog posts or presentations

Remember: The goal isn't just to organize bookmarks—it's to transform passive consumption into active knowledge building and concrete action.