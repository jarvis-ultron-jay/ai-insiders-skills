# Deep Search — Multi-Source AI Research That Goes Beyond Google

Get comprehensive research reports by aggregating results from Brave, Exa, Tavily, and Grok. Stop settling for surface-level answers — get the full picture with source verification and confidence ratings.

## The Problem with Single-Source Search

Every search engine has blind spots:

- **Google/Bing** — Algorithm biases, SEO manipulation, missing academic content
- **Academic databases** — Paywalls, narrow scope, outdated information  
- **Social media** — Noise, misinformation, lack of verification
- **AI assistants** — Hallucination risk, no source transparency

## The Deep Search Solution

This skill orchestrates multiple search APIs to deliver research-grade results:

- **Multi-source aggregation** — Brave + Exa + Tavily + Grok working together
- **Intent-aware scoring** — Different sources weighted by query type
- **Cross-verification** — Facts checked across independent sources
- **Confidence ratings** — Clear indicators of information reliability
- **Comprehensive reports** — Executive summaries plus detailed analysis
- **Source transparency** — Every claim linked to original sources

## Features

- **Parallel Search** — Query multiple engines simultaneously
- **Smart Scoring** — Authority, relevance, recency, and uniqueness metrics  
- **Bias Detection** — Flag potential misinformation and source limitations
- **Academic Focus** — Prioritize scholarly sources when appropriate
- **Real-time Data** — Include breaking news and current information
- **Structured Output** — Professional research report format
- **Source Bibliography** — Complete citation list for all references

## Install

```bash
npx clawhub install deep-search
```

## Setup

Requires API keys for multiple search services:

```bash
# Get API keys from:
# Brave: https://api.search.brave.com/  
# Exa: https://exa.ai/
# Tavily: https://tavily.com/
# Grok: https://console.groq.com/

export BRAVE_API_KEY="your_brave_key"
export EXA_API_KEY="your_exa_key" 
export TAVILY_API_KEY="your_tavily_key"
export GROK_API_KEY="your_grok_key"
```

## Usage Examples

Tell your AI assistant what you need to research:

| Research Type | Example prompts |
|--------------|----------------|
| Academic Research | "Deep search on machine learning bias in healthcare", "Research climate change impact on agriculture" |
| Market Analysis | "Comprehensive research on AI startup funding trends", "Deep dive into SaaS pricing strategies" |
| Fact Checking | "Verify claims about vaccine effectiveness", "Cross-check renewable energy statistics" |
| Technical Research | "Research blockchain scalability solutions", "Deep search on quantum computing applications" |
| Current Events | "Comprehensive analysis of recent trade policy changes", "Research latest developments in space exploration" |
| Competitive Analysis | "Deep search on competitor pricing and features", "Research industry best practices" |

## Output Example

```
🔍 Deep Search Report: AI Safety Regulations
📅 Generated: 2024-01-15 14:23
🌐 Sources: 47 from 4 search engines

📋 Key Findings:
• EU AI Act passed final vote with 71% support (High confidence)
• US considering federal AI oversight agency by Q2 2024 (Medium confidence)  
• Industry self-regulation initiatives show mixed effectiveness (High confidence)

⚡ Quick Answer: AI safety regulation is rapidly evolving globally, with the EU leading on comprehensive legislation while the US focuses on sector-specific approaches.

## Core Information

### Primary Sources
1. **European Parliament AI Act Final Text** - europa.eu (2024-01-12)
   - Authority: 10/10 | Relevance: 10/10 | Recency: 10/10
   - Key Points: 
     • Final vote results and implementation timeline
     • Risk-based classification system details
     • Compliance requirements for high-risk AI systems
   - https://europa.eu/ai-act-final-text

2. **NIST AI Risk Management Framework** - nist.gov (2024-01-08)
   - Authority: 9/10 | Relevance: 9/10 | Recency: 9/10
   - Key Points:
     • Voluntary guidelines for AI system development
     • Risk assessment methodologies
     • Industry adoption metrics
   - https://nist.gov/ai-risk-framework

## Synthesis & Analysis

### Areas of Consensus
- Need for risk-based regulatory approaches
- Importance of transparency and explainability
- Focus on high-risk applications (healthcare, finance, transportation)

### Points of Debate  
- Optimal balance between innovation and safety
- Effectiveness of self-regulation vs mandatory compliance
- International coordination and standard harmonization

### Confidence Assessment
- High Confidence: Regulatory timeline and framework details
- Medium Confidence: Industry impact predictions
- Low Confidence: Long-term effectiveness projections

## Complete Source List
**Regulatory Sources:** 12
**Academic Papers:** 15  
**Industry Reports:** 11
**News Analysis:** 9

Total unique sources consulted: 47
```

## Search Modes

### Academic Research Mode
- Prioritizes peer-reviewed papers and scholarly sources
- Cross-references findings across multiple studies  
- Includes methodology analysis and limitations
- Focuses on citation quality and academic authority

### Breaking News Mode
- Emphasizes real-time information from multiple news sources
- Tracks story development and updates
- Verifies claims across independent outlets
- Flags unconfirmed reports and developing details

### Technical Deep Dive Mode
- Seeks official documentation and specifications
- Includes expert analysis and implementation examples
- Cross-references technical claims with multiple authorities
- Provides practical context and real-world applications

### Market Research Mode  
- Combines analyst reports, industry data, and company filings
- Includes competitive landscape analysis
- Cross-references market size and growth projections
- Verifies financial and performance metrics

### Fact-Checking Mode
- Multi-source verification for all claims
- Seeks authoritative and independent confirmation
- Flags potential misinformation or bias
- Provides clear confidence levels for each assertion

## Quality Guarantees

- **Minimum 3 sources** for any major claim
- **Authority scoring** based on domain expertise and reputation
- **Recency weighting** appropriate to query type
- **Bias flagging** when sources show clear perspective limitations
- **Confidence ratings** for every piece of information
- **Cross-verification** for all quantitative data

## Use Cases

### Researchers & Academics
- Literature reviews that span multiple databases
- Cross-disciplinary research synthesis
- Grant proposal background research
- Hypothesis validation across sources

### Business Professionals
- Market research and competitive intelligence
- Due diligence for partnerships or investments
- Industry trend analysis and forecasting
- Strategic planning research support

### Journalists & Fact-Checkers
- Source verification and cross-referencing
- Background research for investigative stories
- Real-time fact-checking during breaking news
- Expert source identification and verification

### Students & Educators
- Comprehensive research for academic papers
- Multi-perspective analysis of complex topics
- Primary source identification and verification
- Research methodology and bias analysis

## Based On

This skill is inspired by the open-source [deep-search](https://github.com/blessonism/deep-search) project by blessonism (410 GitHub stars), which pioneered multi-source search aggregation with AI-powered synthesis.

---

**Stop searching. Start researching.**

Get the complete picture with sources you can trust.