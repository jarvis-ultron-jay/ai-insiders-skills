---
name: ai-readiness-assessment
description: Build an AI readiness assessment tool scoring users across 5 dimensions with time/money savings projections, tier labels, Supabase storage, live Impact Wall scoreboard, and GHL lead capture. Client-side scoring, zero API calls.
---
# AI Readiness Assessment

**Version:** 1.0.0
**Description:** Build and operate an AI readiness assessment tool that scores users across 5 dimensions (Awareness, Integration, Productivity, Personal Usage, Future Readiness), calculates time and money savings projections, assigns tier labels, stores results in Supabase, displays a live "Impact Wall" scoreboard, and integrates with GoHighLevel (GHL) for lead capture. The scoring runs entirely client-side (zero API calls) with static HTML/JS hosted on Vercel.

**Trigger phrases:** "AI readiness", "readiness assessment", "AI assessment", "AI quiz", "AI score", "impact wall", "readiness tool", "AI survey", "readiness score", "AI readiness build", "assessment tool"

---

## Context: Purpose of the AI Readiness Assessment

Wild Success offers AI coaching programmes. To qualify leads and demonstrate value, the AI Readiness Assessment:
1. **Scores** a user's current AI adoption across 5 dimensions
2. **Calculates** personalised time and money savings estimates
3. **Assigns** a tier label (Novice → Native) that frames their current position
4. **Stores** results for analytics and follow-up
5. **Displays** an "Impact Wall" showing aggregate results (social proof)
6. **Captures** the lead in GHL for sales follow-up

The tool must be lightweight (no server-side scoring), fast (< 2s page loads), and mobile-friendly.

---

## Section 1: Question Design

### 1.1 The 5 Dimensions

| # | Dimension | What It Measures | Questions |
|---|-----------|-----------------|-----------|
| 1 | **Awareness** | Knowledge of AI tools, trends, terminology | Q1, Q2, Q3 |
| 2 | **Integration** | How deeply AI is embedded in their work/business | Q4, Q5, Q6 |
| 3 | **Productivity** | Actual time savings and efficiency gains from AI | Q7, Q8, Q9 |
| 4 | **Personal Usage** | Daily personal use of AI tools | Q10, Q11, Q12 |
| 5 | **Future Readiness** | Preparedness for AI-driven changes in their industry | Q13, Q14, Q15 |

### 1.2 The 15 Questions

Each question scores 0-4 (5 options from least to most AI-ready).

#### Dimension 1: Awareness

**Q1. How familiar are you with current AI tools and platforms?**
- (0) I've heard of AI but don't really know what tools are available
- (1) I know about ChatGPT and a few others but haven't explored much
- (2) I'm aware of several AI tools and have tried a couple
- (3) I regularly follow AI developments and know many tools by category
- (4) I have deep knowledge of AI tools across multiple categories and use cases

**Q2. How would you describe your understanding of how AI works?**
- (0) I have no idea how AI works
- (1) I have a very basic understanding (it learns from data somehow)
- (2) I understand the basics of machine learning and large language models
- (3) I can explain AI concepts clearly and understand its limitations
- (4) I have a technical understanding and can evaluate AI models/approaches

**Q3. How often do you stay updated on AI news and developments?**
- (0) Never — I don't follow AI news
- (1) Rarely — maybe see something on social media occasionally
- (2) Monthly — I read articles or watch videos about AI
- (3) Weekly — I actively follow AI newsletters, podcasts, or communities
- (4) Daily — AI developments are part of my regular information diet

#### Dimension 2: Integration

**Q4. How is AI currently used in your business or work?**
- (0) Not at all — we don't use any AI tools
- (1) Minimal — maybe one person uses ChatGPT occasionally
- (2) Some tools — a few team members use AI for specific tasks
- (3) Significant — AI is integrated into several workflows and processes
- (4) Extensive — AI is a core part of our operations and strategy

**Q5. How would you rate your organisation's AI strategy?**
- (0) We don't have one and haven't thought about it
- (1) We've discussed it but have no formal plan
- (2) We have a basic plan but limited execution
- (3) We have a clear strategy with ongoing implementation
- (4) AI strategy is embedded in our business strategy with dedicated resources

**Q6. How many AI tools does your team actively use?**
- (0) None
- (1) 1-2 tools
- (2) 3-5 tools
- (3) 6-10 tools
- (4) 10+ tools across multiple categories

#### Dimension 3: Productivity

**Q7. How many hours per week do you spend on tasks that AI could automate?**
- (0) I'm not sure / I don't think AI could help
- (1) 1-3 hours
- (2) 4-7 hours
- (3) 8-15 hours
- (4) 16+ hours

*Note: This question is used directly in the time savings calculation (see §2.3).*

**Q8. How much has AI already improved your productivity?**
- (0) It hasn't — I don't use AI for work
- (1) Slightly — saves me a few minutes here and there
- (2) Moderately — saves me 1-2 hours per week
- (3) Significantly — saves me 3-5 hours per week
- (4) Transformatively — saves me 5+ hours per week and changed how I work

**Q9. What types of tasks do you currently use AI for?**
- (0) None
- (1) Basic tasks (writing emails, simple questions)
- (2) Content creation (social media, blog posts, marketing copy)
- (3) Analysis and strategy (data analysis, market research, planning)
- (4) Complex workflows (automation chains, code, system integration)

#### Dimension 4: Personal Usage

**Q10. How often do you personally use AI tools?**
- (0) Never
- (1) A few times a month
- (2) A few times a week
- (3) Daily
- (4) Multiple times per day — it's my default starting point

**Q11. Which best describes your relationship with AI tools?**
- (0) I avoid them — they feel overwhelming or unnecessary
- (1) I'm curious but haven't developed a habit
- (2) I use them when I remember to, but it's not automatic
- (3) They're part of my daily workflow — I reach for AI naturally
- (4) I can't imagine working without them — they're essential to my process

**Q12. How comfortable are you learning new AI tools?**
- (0) Very uncomfortable — technology frustrates me
- (1) Somewhat uncomfortable — I need a lot of guidance
- (2) Neutral — I can figure things out with some help
- (3) Comfortable — I enjoy exploring new tools
- (4) Very comfortable — I actively seek out and test new AI tools

#### Dimension 5: Future Readiness

**Q13. How prepared do you feel for AI-driven changes in your industry?**
- (0) Not at all — I haven't thought about it
- (1) Slightly — I know changes are coming but feel unprepared
- (2) Somewhat — I've started learning but have gaps
- (3) Well prepared — I have a plan and am actively upskilling
- (4) Very prepared — I'm ahead of the curve and helping others adapt

**Q14. How do you view AI's impact on your career/business in the next 2 years?**
- (0) Irrelevant — it won't affect me
- (1) Minor inconvenience — some adjustments needed
- (2) Moderate impact — I'll need to adapt some processes
- (3) Significant opportunity — AI will create new possibilities
- (4) Transformative — AI will fundamentally reshape what I do (and I'm ready)

**Q15. What's your biggest barrier to using AI more effectively?**
- (0) I don't see the value — it doesn't seem relevant to me
- (1) Overwhelm — too many tools, don't know where to start
- (2) Time — I know I should learn but haven't prioritised it
- (3) Depth — I use basic features but want to go deeper
- (4) Nothing — I'm already maximising AI in my work and looking for the next level

---

## Section 2: Scoring Model

### 2.1 Raw Score Calculation

```
Raw Score = sum of all 15 answers (each 0-4)
Minimum: 0 (all zeros)
Maximum: 60 (all fours)
```

### 2.2 Scaled Score (0-100)

```javascript
function calculateScaledScore(rawScore) {
  // Linear scaling: 0-60 raw → 0-100 scaled
  return Math.round((rawScore / 60) * 100);
}
```

### 2.3 Dimension Scores

Each dimension has 3 questions (max 12 points per dimension). Calculate percentage:

```javascript
function calculateDimensionScores(answers) {
  const dimensions = {
    awareness: { questions: [0, 1, 2], label: 'Awareness' },
    integration: { questions: [3, 4, 5], label: 'Integration' },
    productivity: { questions: [6, 7, 8], label: 'Productivity' },
    personal_usage: { questions: [9, 10, 11], label: 'Personal Usage' },
    future_readiness: { questions: [12, 13, 14], label: 'Future Readiness' }
  };
  
  const results = {};
  for (const [key, dim] of Object.entries(dimensions)) {
    const sum = dim.questions.reduce((total, qIdx) => total + answers[qIdx], 0);
    const maxPossible = dim.questions.length * 4;  // 12
    results[key] = {
      label: dim.label,
      raw: sum,
      percentage: Math.round((sum / maxPossible) * 100)
    };
  }
  return results;
}
```

### 2.4 Tier Assignment

| Overall Score | Tier | Label | Description |
|--------------|------|-------|-------------|
| 0-20 | 1 | **Novice** | Just starting to explore AI. Massive opportunity ahead. |
| 21-40 | 2 | **Curious** | Aware and interested but not yet leveraging AI consistently. |
| 41-60 | 3 | **Adopter** | Actively using AI with room to deepen integration. |
| 61-80 | 4 | **Integrated** | AI is a core part of your workflow. Focus on optimisation. |
| 81-100 | 5 | **Native** | AI-native operator. You're leading the charge. |

```javascript
function assignTier(scaledScore) {
  if (scaledScore <= 20) return { tier: 1, label: 'Novice', emoji: '🌱' };
  if (scaledScore <= 40) return { tier: 2, label: 'Curious', emoji: '🔍' };
  if (scaledScore <= 60) return { tier: 3, label: 'Adopter', emoji: '⚡' };
  if (scaledScore <= 80) return { tier: 4, label: 'Integrated', emoji: '🚀' };
  return { tier: 5, label: 'Native', emoji: '🧠' };
}
```

### 2.5 Time Savings Calculation

Based on Q7 (manual hours that AI could automate) and the user's dimension scores.

```javascript
function calculateTimeSavings(answers, dimensionScores) {
  // Q7 maps to estimated automatable hours per week
  const q7HoursMap = {
    0: 2,    // "Not sure" — conservative estimate
    1: 2,    // 1-3 hours → midpoint 2
    2: 5.5,  // 4-7 hours → midpoint 5.5
    3: 11.5, // 8-15 hours → midpoint 11.5
    4: 20    // 16+ hours → conservative 20
  };
  
  const baseHours = q7HoursMap[answers[6]];  // Q7 is index 6 (0-indexed)
  
  // Dimension multipliers: higher existing integration = less room for savings
  // Lower integration scores = more room for improvement
  const integrationScore = dimensionScores.integration.percentage;
  const productivityScore = dimensionScores.productivity.percentage;
  
  // If someone is already highly integrated (80%+), they've captured most savings
  // If they're low (0-20%), there's huge room for improvement
  const improvementMultiplier = 1 - (integrationScore / 200);  // Range: 0.6 to 1.0
  
  // Estimated weekly time savings (hours)
  const timeSavedWeekly = Math.round(baseHours * improvementMultiplier * 10) / 10;
  
  return {
    currentManualHours: baseHours,
    timeSavedWeekly: timeSavedWeekly,
    timeSavedMonthly: Math.round(timeSavedWeekly * 4.33 * 10) / 10,
    timeSavedYearly: Math.round(timeSavedWeekly * 48 * 10) / 10  // 48 working weeks
  };
}
```

### 2.6 Money Savings Calculation

```javascript
function calculateMoneySavings(timeSavings, hourlyRate) {
  // hourlyRate provided by user in the assessment form
  return {
    weeklySavings: Math.round(timeSavings.timeSavedWeekly * hourlyRate),
    monthlySavings: Math.round(timeSavings.timeSavedMonthly * hourlyRate),
    annualSavings: Math.round(timeSavings.timeSavedWeekly * hourlyRate * 48)
  };
}
```

**Hourly rate input:** Ask the user for their approximate hourly rate (or annual salary / 2,000 hours). This personalises the money savings figure.

---

## Section 3: Tech Stack & Implementation

### 3.1 Architecture

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Static HTML/JS  │────▶│    Supabase      │     │    GHL Webhook   │
│  (Vercel)        │     │    (Storage)     │     │    (Lead Capture)│
│                  │     │                  │     │                  │
│  • Assessment    │     │  • Scores table  │     │  • Contact create│
│  • Scoring       │     │  • Impact data   │     │  • Tag apply     │
│  • Results page  │     │                  │     │                  │
│  • Impact Wall   │◀────│                  │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
```

**Key design decision:** All scoring logic runs client-side in JavaScript. Zero API calls for calculation. This means:
- Instant results (no server roundtrip)
- Works offline after page load
- No scoring API to maintain or pay for
- Results are stored to Supabase AFTER display (non-blocking)

### 3.2 File Structure

```
ai-readiness-assessment/
  index.html          # Assessment landing / Impact Wall
  assessment.html     # The 15-question assessment
  results.html        # Personal results page
  css/
    styles.css        # All styles
  js/
    scoring.js        # Scoring logic (all functions from §2)
    assessment.js     # Form handling, progression, validation
    results.js        # Results display, chart rendering
    impact-wall.js    # Supabase query + display for Impact Wall
    supabase.js       # Supabase client config
  assets/
    logo.svg          # Wild Success logo
    og-image.png      # Social sharing image
```

### 3.3 Assessment Flow

```
1. Impact Wall (index.html)
   └── "Take the Assessment" CTA
        │
2. Assessment (assessment.html)
   ├── User info: first name, last initial, email, location, user type, hourly rate
   ├── 15 questions (one at a time or paginated in groups of 3)
   └── "Get Your Results" button
        │
3. Results (results.html)
   ├── Overall score + tier badge
   ├── Dimension radar chart
   ├── Time savings projection
   ├── Money savings projection
   ├── Personalised recommendations
   └── CTA: "Join the AI Community" → Circle invite link
        │
   [Background: store results to Supabase + fire GHL webhook]
```

### 3.4 Assessment Form Fields

**Pre-assessment (user info):**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| first_name | text | Yes | Displayed on Impact Wall |
| last_initial | text | Yes | Privacy (shows "John S." not "John Smith") |
| email | email | Yes | For GHL lead capture + deduplication |
| location | select | Yes | Country or region (for Impact Wall geography) |
| user_type | select | Yes | Options: Business Owner, Employee, Freelancer, Student, Other |
| hourly_rate | number | Yes | For money savings calculation. Prompt: "What's your approximate hourly rate (or annual salary ÷ 2,000)?" |

### 3.5 Client-Side Scoring Implementation

```html
<!-- assessment.html — simplified -->
<form id="assessment-form">
  <!-- User info fields -->
  <input type="text" id="first_name" required>
  <input type="text" id="last_initial" required>
  <input type="email" id="email" required>
  <select id="location" required>
    <option value="AU">Australia</option>
    <option value="US">United States</option>
    <option value="UK">United Kingdom</option>
    <!-- ... more countries -->
  </select>
  <select id="user_type" required>
    <option value="business_owner">Business Owner</option>
    <option value="employee">Employee</option>
    <option value="freelancer">Freelancer</option>
    <option value="student">Student</option>
    <option value="other">Other</option>
  </select>
  <input type="number" id="hourly_rate" min="1" required>
  
  <!-- 15 questions (Q1-Q15) -->
  <!-- Each question is a radio group with values 0-4 -->
  <div class="question" data-q="0">
    <p>Q1. How familiar are you with current AI tools and platforms?</p>
    <label><input type="radio" name="q0" value="0"> I've heard of AI but...</label>
    <label><input type="radio" name="q0" value="1"> I know about ChatGPT...</label>
    <label><input type="radio" name="q0" value="2"> I'm aware of several...</label>
    <label><input type="radio" name="q0" value="3"> I regularly follow...</label>
    <label><input type="radio" name="q0" value="4"> I have deep knowledge...</label>
  </div>
  <!-- ... Q2 through Q15 ... -->
  
  <button type="submit">Get Your Results</button>
</form>

<script src="js/scoring.js"></script>
<script src="js/assessment.js"></script>
```

```javascript
// js/assessment.js
document.getElementById('assessment-form').addEventListener('submit', function(e) {
  e.preventDefault();
  
  // Collect answers
  const answers = [];
  for (let i = 0; i < 15; i++) {
    const selected = document.querySelector(`input[name="q${i}"]:checked`);
    if (!selected) {
      alert(`Please answer question ${i + 1}`);
      return;
    }
    answers.push(parseInt(selected.value));
  }
  
  // Collect user info
  const userInfo = {
    first_name: document.getElementById('first_name').value,
    last_initial: document.getElementById('last_initial').value,
    email: document.getElementById('email').value,
    location: document.getElementById('location').value,
    user_type: document.getElementById('user_type').value,
    hourly_rate: parseFloat(document.getElementById('hourly_rate').value)
  };
  
  // Calculate all scores (client-side, zero API calls)
  const rawScore = answers.reduce((sum, a) => sum + a, 0);
  const scaledScore = calculateScaledScore(rawScore);
  const dimensionScores = calculateDimensionScores(answers);
  const tier = assignTier(scaledScore);
  const timeSavings = calculateTimeSavings(answers, dimensionScores);
  const moneySavings = calculateMoneySavings(timeSavings, userInfo.hourly_rate);
  
  // Package results
  const results = {
    ...userInfo,
    raw_score: rawScore,
    overall_score: scaledScore,
    tier: tier.label,
    tier_number: tier.tier,
    dim_awareness: dimensionScores.awareness.percentage,
    dim_integration: dimensionScores.integration.percentage,
    dim_productivity: dimensionScores.productivity.percentage,
    dim_personal_usage: dimensionScores.personal_usage.percentage,
    dim_future_readiness: dimensionScores.future_readiness.percentage,
    time_saved_weekly: timeSavings.timeSavedWeekly,
    annual_savings: moneySavings.annualSavings,
    answers: answers
  };
  
  // Store in sessionStorage for results page
  sessionStorage.setItem('assessment_results', JSON.stringify(results));
  
  // Navigate to results page
  window.location.href = 'results.html';
});
```

---

## Section 4: Data Model (Supabase)

### 4.1 ai_readiness_scores Table

```sql
CREATE TABLE ai_readiness_scores (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  first_name TEXT NOT NULL,
  last_initial TEXT NOT NULL,
  email TEXT NOT NULL,
  location TEXT,
  user_type TEXT,
  hourly_rate NUMERIC,
  overall_score INTEGER NOT NULL CHECK (overall_score >= 0 AND overall_score <= 100),
  tier TEXT NOT NULL CHECK (tier IN ('Novice', 'Curious', 'Adopter', 'Integrated', 'Native')),
  tier_number INTEGER NOT NULL CHECK (tier_number >= 1 AND tier_number <= 5),
  raw_score INTEGER,
  dim_awareness INTEGER CHECK (dim_awareness >= 0 AND dim_awareness <= 100),
  dim_integration INTEGER CHECK (dim_integration >= 0 AND dim_integration <= 100),
  dim_productivity INTEGER CHECK (dim_productivity >= 0 AND dim_productivity <= 100),
  dim_personal_usage INTEGER CHECK (dim_personal_usage >= 0 AND dim_personal_usage <= 100),
  dim_future_readiness INTEGER CHECK (dim_future_readiness >= 0 AND dim_future_readiness <= 100),
  time_saved_weekly NUMERIC,
  annual_savings NUMERIC,
  answers JSONB,          -- [0,2,3,1,4,...] — raw answer values for each question
  source TEXT DEFAULT 'web',  -- 'web', 'event', 'email'
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_readiness_email ON ai_readiness_scores(email);
CREATE INDEX idx_readiness_tier ON ai_readiness_scores(tier);
CREATE INDEX idx_readiness_created ON ai_readiness_scores(created_at);

-- Enable Row Level Security
ALTER TABLE ai_readiness_scores ENABLE ROW LEVEL SECURITY;

-- Public can INSERT (for assessment submissions)
CREATE POLICY "Allow public insert" ON ai_readiness_scores
  FOR INSERT WITH CHECK (true);

-- Public can read aggregated data (for Impact Wall)
-- Actual RLS policy depends on your needs — may use anon key with limited SELECT
CREATE POLICY "Allow public read for impact wall" ON ai_readiness_scores
  FOR SELECT USING (true);
```

### 4.2 Storing Results

```javascript
// js/supabase.js
import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js/+esm';

const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...';  // Anon key (public, limited by RLS)

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
```

```javascript
// js/results.js — called on results.html load
async function saveResults(results) {
  const { error } = await supabase
    .from('ai_readiness_scores')
    .insert({
      first_name: results.first_name,
      last_initial: results.last_initial,
      email: results.email,
      location: results.location,
      user_type: results.user_type,
      hourly_rate: results.hourly_rate,
      overall_score: results.overall_score,
      tier: results.tier,
      tier_number: results.tier_number,
      raw_score: results.raw_score,
      dim_awareness: results.dim_awareness,
      dim_integration: results.dim_integration,
      dim_productivity: results.dim_productivity,
      dim_personal_usage: results.dim_personal_usage,
      dim_future_readiness: results.dim_future_readiness,
      time_saved_weekly: results.time_saved_weekly,
      annual_savings: results.annual_savings,
      answers: results.answers
    });
  
  if (error) {
    console.error('Failed to save results:', error);
    // Non-blocking — user still sees their results
  }
}
```

---

## Section 5: Impact Wall

### 5.1 What the Impact Wall Shows

The Impact Wall is a live scoreboard displayed on the assessment landing page. It pulls real data from Supabase and shows:

1. **Total assessments completed** (counter)
2. **Average score** across all participants
3. **Tier distribution** (pie or bar chart)
4. **Total collective time saved** (sum of all time_saved_weekly × 48)
5. **Total collective money saved** (sum of all annual_savings)
6. **Recent completions** (scrolling feed of "John S. from Australia scored 72 — Integrated 🚀")
7. **Geographic distribution** (by country)

### 5.2 Impact Wall Queries

```javascript
// js/impact-wall.js

async function loadImpactWallData() {
  // Total count + averages
  const { data: stats, error: statsErr } = await supabase
    .rpc('impact_wall_stats');  // Or use direct queries below
  
  // If not using RPC, query directly:
  
  // 1. Total assessments
  const { count: totalCount } = await supabase
    .from('ai_readiness_scores')
    .select('*', { count: 'exact', head: true });
  
  // 2. Average score
  const { data: avgData } = await supabase
    .from('ai_readiness_scores')
    .select('overall_score');
  const avgScore = avgData.reduce((s, r) => s + r.overall_score, 0) / avgData.length;
  
  // 3. Tier distribution
  const { data: tierData } = await supabase
    .from('ai_readiness_scores')
    .select('tier')
    .then(({ data }) => {
      const counts = {};
      data.forEach(r => { counts[r.tier] = (counts[r.tier] || 0) + 1; });
      return { data: counts };
    });
  
  // 4. Collective savings
  const { data: savingsData } = await supabase
    .from('ai_readiness_scores')
    .select('time_saved_weekly, annual_savings');
  const totalTimeSavedYearly = savingsData.reduce((s, r) => 
    s + (r.time_saved_weekly * 48), 0);
  const totalMoneySaved = savingsData.reduce((s, r) => 
    s + r.annual_savings, 0);
  
  // 5. Recent completions (last 10)
  const { data: recent } = await supabase
    .from('ai_readiness_scores')
    .select('first_name, last_initial, location, overall_score, tier, created_at')
    .order('created_at', { ascending: false })
    .limit(10);
  
  return {
    totalCount,
    avgScore: Math.round(avgScore),
    tierDistribution: tierData,
    totalTimeSavedYearly: Math.round(totalTimeSavedYearly),
    totalMoneySaved: Math.round(totalMoneySaved),
    recentCompletions: recent
  };
}
```

### 5.3 Impact Wall Display

```html
<!-- index.html — Impact Wall section -->
<section class="impact-wall">
  <h2>AI Readiness Across Our Community</h2>
  
  <div class="stats-grid">
    <div class="stat-card">
      <span class="stat-number" id="total-count">—</span>
      <span class="stat-label">Assessments Completed</span>
    </div>
    <div class="stat-card">
      <span class="stat-number" id="avg-score">—</span>
      <span class="stat-label">Average Score</span>
    </div>
    <div class="stat-card">
      <span class="stat-number" id="total-hours">—</span>
      <span class="stat-label">Hours Saved Per Year (Collectively)</span>
    </div>
    <div class="stat-card">
      <span class="stat-number" id="total-savings">—</span>
      <span class="stat-label">Annual Savings Unlocked</span>
    </div>
  </div>
  
  <div class="tier-chart" id="tier-chart">
    <!-- Render tier distribution chart here -->
  </div>
  
  <div class="recent-feed" id="recent-feed">
    <!-- Scrolling recent completions -->
    <!-- "Sarah M. from UK scored 65 — Integrated 🚀" -->
  </div>
  
  <a href="assessment.html" class="cta-button">Take the Assessment →</a>
</section>
```

### 5.4 Animating Counters

For social proof impact, animate the numbers counting up:

```javascript
function animateCounter(element, targetValue, duration = 2000, prefix = '', suffix = '') {
  let start = 0;
  const increment = targetValue / (duration / 16);  // ~60fps
  
  function update() {
    start += increment;
    if (start >= targetValue) {
      element.textContent = prefix + targetValue.toLocaleString() + suffix;
      return;
    }
    element.textContent = prefix + Math.round(start).toLocaleString() + suffix;
    requestAnimationFrame(update);
  }
  update();
}

// Usage
animateCounter(document.getElementById('total-count'), 1247);
animateCounter(document.getElementById('avg-score'), 48, 2000, '', '/100');
animateCounter(document.getElementById('total-hours'), 89400, 2500, '', ' hrs');
animateCounter(document.getElementById('total-savings'), 4250000, 3000, '$', '');
```

---

## Section 6: GHL Integration (Lead Capture)

### 6.1 Webhook on Completion

When a user completes the assessment, fire a webhook to GHL to create/update the contact:

```javascript
async function sendToGHL(results) {
  const webhookUrl = 'https://services.leadconnectorhq.com/hooks/<WEBHOOK_ID>';
  // Alternatively, use a GHL form webhook or Zapier/Make webhook
  
  const payload = {
    first_name: results.first_name,
    last_name: results.last_initial,
    email: results.email,
    country: results.location,
    tags: [
      'ai-readiness-assessment',
      `ai-tier-${results.tier.toLowerCase()}`,
      `ai-score-${results.overall_score}`
    ],
    custom_fields: {
      ai_readiness_score: results.overall_score,
      ai_readiness_tier: results.tier,
      ai_time_saved_weekly: results.time_saved_weekly,
      ai_annual_savings: results.annual_savings
    }
  };
  
  try {
    await fetch(webhookUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });
  } catch (error) {
    console.error('GHL webhook failed:', error);
    // Non-blocking — don't break the user experience
  }
}
```

### 6.2 GHL Tags Applied

| Tag | Purpose |
|-----|---------|
| `ai-readiness-assessment` | Identifies assessment completers |
| `ai-tier-novice` through `ai-tier-native` | Tier-based segmentation |
| `ai-score-72` | Exact score for filtering |

### 6.3 Follow-Up Automation

In GHL, set up a workflow triggered by the `ai-readiness-assessment` tag:
1. Wait 5 minutes
2. Send personalised email based on tier:
   - **Novice/Curious:** "Start your AI journey — free resources"
   - **Adopter:** "Level up your AI integration — workshop invite"
   - **Integrated/Native:** "Join our AI community — Circle invite"
3. Add to nurture sequence

---

## Section 7: Results Page

### 7.1 Results Display Components

1. **Overall Score Badge** — Large circle with score and tier label
2. **Dimension Radar Chart** — 5-axis radar showing dimension percentages
3. **Time Savings Card** — "You could save X hours per week with better AI integration"
4. **Money Savings Card** — "That's $X per year at your rate of $Y/hour"
5. **Personalised Recommendations** — Based on lowest-scoring dimensions
6. **CTA** — "Join the AI Insiders Community" → Circle invite link

### 7.2 Personalised Recommendations

```javascript
function generateRecommendations(dimensionScores) {
  const recommendations = [];
  const sorted = Object.entries(dimensionScores)
    .sort((a, b) => a[1].percentage - b[1].percentage);
  
  // Focus on the 2 weakest dimensions
  const weakest = sorted.slice(0, 2);
  
  const recMap = {
    awareness: "Start with our free AI Tools Guide — discover the top 20 AI tools sorted by use case.",
    integration: "Book a strategy session to map AI integration points in your business workflow.",
    productivity: "Join our next AI Productivity Workshop — learn to automate your highest-value tasks.",
    personal_usage: "Download our 7-Day AI Challenge — build a daily AI habit one tool at a time.",
    future_readiness: "Read our AI Industry Impact Report — understand what's coming and how to prepare."
  };
  
  for (const [key, scores] of weakest) {
    if (recMap[key]) {
      recommendations.push({
        dimension: scores.label,
        score: scores.percentage,
        recommendation: recMap[key]
      });
    }
  }
  
  return recommendations;
}
```

### 7.3 CTA Flow

```
Results Page → "Join 56,000+ in the Wild Coaching Community" → Circle invite link
                                                              (or GHL form for capture)
```

The CTA should be contextual based on tier:
- **Novice/Curious:** "Start your AI journey with free resources" → Lead magnet
- **Adopter:** "Level up with our AI programme" → Programme landing page
- **Integrated/Native:** "Join the AI Insiders community" → Circle invite

---

## Section 8: Deployment

### 8.1 Vercel Deployment

```bash
cd /path/to/ai-readiness-assessment

# Deploy as static site
npx vercel --yes --prod

# Or for framework detection:
# Vercel will auto-detect static HTML and serve accordingly
```

### 8.2 Environment Variables

For the Supabase client (these are public/anon keys — safe for client-side):

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
```

For the GHL webhook URL, hardcode in the JS (it's a webhook endpoint, not a secret) or use an environment variable if using a build step.

### 8.3 Custom Domain (Optional)

```
ai.wildsuccess.global → Vercel deployment
```

---

## Section 9: Analytics & Reporting

### 9.1 Key Metrics to Track

| Metric | Query | Purpose |
|--------|-------|---------|
| Total completions | `COUNT(*)` | Volume |
| Average score | `AVG(overall_score)` | Community baseline |
| Tier distribution | `COUNT(*) GROUP BY tier` | Segmentation |
| Completion rate | Page views → submissions | Funnel health |
| Geographic distribution | `COUNT(*) GROUP BY location` | Market insights |
| Score by user_type | `AVG(overall_score) GROUP BY user_type` | Segment insights |
| Total projected savings | `SUM(annual_savings)` | Social proof metric |

### 9.2 Reporting Query

```sql
-- Monthly summary report
SELECT
  DATE_TRUNC('month', created_at) AS month,
  COUNT(*) AS completions,
  ROUND(AVG(overall_score), 1) AS avg_score,
  COUNT(*) FILTER (WHERE tier = 'Novice') AS novice_count,
  COUNT(*) FILTER (WHERE tier = 'Curious') AS curious_count,
  COUNT(*) FILTER (WHERE tier = 'Adopter') AS adopter_count,
  COUNT(*) FILTER (WHERE tier = 'Integrated') AS integrated_count,
  COUNT(*) FILTER (WHERE tier = 'Native') AS native_count,
  ROUND(SUM(annual_savings)) AS total_projected_savings,
  ROUND(SUM(time_saved_weekly * 48)) AS total_hours_saved_yearly
FROM ai_readiness_scores
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month DESC;
```

---

## Known Gotchas

| Issue | Impact | Prevention |
|-------|--------|------------|
| Supabase anon key exposed in client JS | Anyone can call the API | RLS policies restrict to INSERT only + limited SELECT for Impact Wall |
| User submits multiple times | Duplicate entries | Deduplicate on email — upsert or check before insert |
| Hourly rate = 0 or unrealistic | Money savings calculation broken | Validate input: min $10/hr, max $1,000/hr |
| Assessment abandoned mid-way | Incomplete data | Don't save until completion. Track form progression for analytics. |
| Impact Wall slow with many records | Poor UX | Use Supabase RPC for pre-aggregated stats instead of client-side aggregation |
| GHL webhook fails | Lead not captured | Retry once. Log failure. The Supabase record still exists. |

---

*This skill is self-contained. A fresh agent can build and deploy a complete AI Readiness Assessment tool using only this document.*
