# Scholar — Academic Reading That Actually Builds Knowledge

Transform passive paper consumption into active knowledge building. Read papers, extract insights, create connections, and build a networked knowledge base that gets smarter over time.

## The Problem

Most academic reading is passive and forgettable:

- Read papers → Forget key insights within weeks
- No systematic way to connect new knowledge to existing understanding  
- Literature reviews become overwhelming information dumps
- Research questions evolve without tracking the reasoning
- Knowledge stays siloed instead of building compound returns

## The Solution

This skill creates an active reading workflow with Obsidian integration that turns every paper into lasting knowledge:

- **Structured extraction** — Systematic note-taking that captures insights, not just summaries
- **Bi-directional linking** — Connect new papers to existing knowledge network
- **Question tracking** — Follow how research questions evolve and get answered
- **Connection mapping** — Build visual networks of concepts, authors, and ideas
- **Review workflows** — Regular synthesis to identify patterns and gaps

## Features

- **Three-Pass Reading Method** — Efficient paper processing from overview to deep analysis
- **Active Note Templates** — Structured templates for extracting insights and connections
- **Knowledge Network Building** — Link concepts, authors, methods, and findings
- **Literature Review Integration** — Systematic approach to research synthesis
- **Citation Management** — Seamless integration with Zotero and BibTeX
- **Research Question Evolution** — Track how questions develop over time
- **Collaboration Support** — Share insights and coordinate team reading

## Install

```bash
npx clawhub install scholar
```

## Setup

Requires Obsidian for knowledge management:

```bash
# Install Obsidian
brew install --cask obsidian
# Or download from: https://obsidian.md

# Recommended plugins:
# - Citations plugin (Zotero integration)
# - Dataview plugin (dynamic note queries)
# - Templater plugin (automated note templates)
# - Graph view (built-in network visualization)
```

## Usage Examples

Tell your AI assistant what you need:

| Task | Example prompts |
|------|----------------|
| **Paper Analysis** | "Help me read this ML paper systematically", "Extract key insights from this research" |
| **Note Taking** | "Create reading notes for this paper", "Connect this to my existing knowledge on [topic]" |
| **Literature Review** | "Help me synthesize papers on [topic]", "Find patterns across these studies" |
| **Knowledge Mapping** | "Build a concept map of my research area", "Show connections between authors" |
| **Question Tracking** | "Track how my research question evolved", "What questions does this paper raise?" |
| **Research Planning** | "Identify gaps in the literature", "Plan my reading queue for next week" |

## Workflow Overview

### Phase 1: Paper Triage (5 minutes)
Quick assessment to prioritize reading:
- Rate relevance, quality, and novelty (1-10)
- One-sentence summary of main contribution
- Decide: read thoroughly, skim, archive, or discard

### Phase 2: Active Reading (30-180 minutes)
Three-pass method for efficient comprehension:
- **Pass 1:** Overview (10 min) — Title, abstract, figures, conclusions
- **Pass 2:** Focused reading (30-60 min) — Skip proofs, note questions  
- **Pass 3:** Deep dive (1-3 hours) — Full understanding, evaluate evidence

### Phase 3: Knowledge Extraction (15-30 minutes)
Structured note-taking with connection building:
- Extract key insights and methodology
- Identify connections to existing knowledge
- Generate questions and research ideas
- Link to related papers, authors, and concepts

### Phase 4: Network Building (Ongoing)
Continuous knowledge base development:
- Weekly review to identify patterns
- Monthly domain analysis using graph view
- Quarterly research direction planning
- Update concept maps and connection patterns

## Note Structure Example

```markdown
# Deep Learning for Scientific Discovery - Reading Notes

## Key Contributions
1. Novel architecture combining physics constraints with neural networks
2. 40% improvement over previous methods on benchmark datasets
3. Demonstrates transferability across three scientific domains

## Methodology
**Approach:** Physics-informed neural networks with domain adaptation
**Strengths:** Theoretically grounded, empirically validated
**Limitations:** Requires domain expertise, computationally expensive

## Connections
**Builds on:** [[Physics-Informed Networks]] by [[Karniadakis et al.]]
**Contradicts:** [[Pure Data-Driven Approach]] on need for domain knowledge
**Enables:** New research in [[Scientific ML]], [[Climate Modeling]]

## Personal Reflections
**For my research:** Could apply this framework to materials science
**Critical questions:** How sensitive to physics model accuracy?
**Follow-up:** Contact [[lead author]] about parameter sensitivity

## Action Items
- [ ] Read foundational paper on [[Physics-Informed Networks]]
- [ ] Test approach on my dataset
- [ ] Explore collaboration opportunities
```

## Knowledge Network Patterns

### Connection Types
- **Concept → Paper:** `[[Machine Learning]]` appears in `[[Paper Title]]`
- **Author → Ideas:** `[[Researcher Name]]` developed `[[Key Framework]]`  
- **Method → Applications:** `[[Deep Learning]]` used in `[[Paper A]]`, `[[Paper B]]`
- **Theory → Evidence:** `[[Theory X]]` supported by `[[Study 1]]`, contradicted by `[[Study 2]]`

### Network Health Indicators
- **Connection density** — How well-linked are your notes?
- **Concept coverage** — Are key ideas properly networked?
- **Question tracking** — Are research questions systematically addressed?
- **Knowledge gaps** — Where does understanding remain incomplete?

## Advanced Features

### Systematic Literature Review
- Structured search strategy and paper screening
- Quality assessment and bias evaluation  
- Synthesis across multiple studies and methods
- Gap identification and future direction planning

### Research Question Evolution
Track how questions develop as you read:
```markdown
# Research Question Evolution

## Initial: "How does AI affect productivity?"
## Refined: "How do different AI tools impact knowledge worker productivity across industries?"
## Current: "What mediates the relationship between AI tool adoption and productivity gains in professional services?"

**Evolution drivers:** 
- [[Paper A]] showed industry variation matters
- [[Paper B]] revealed adoption ≠ effective use
- [[Meta-analysis C]] highlighted mediating factors
```

### Collaboration Workflows
- Team reading assignments and discussion notes
- Shared insights and cross-pollination of ideas
- Collaborative literature reviews and synthesis
- Knowledge transfer between team members

### Integration with Writing
Direct connection to research paper writing:
- Literature review sections generated from note network
- Citations pulled automatically from linked papers
- Research gaps identified from knowledge base analysis
- Methodology inspired by successful approaches in notes

## Use Cases

### PhD Students & Researchers
- Comprehensive literature mastery for dissertation research
- Network building within academic fields
- Collaboration with advisors and peers
- Preparation for qualifying exams and conferences

### Industry R&D Teams  
- Technology landscape analysis and competitive intelligence
- Cross-functional knowledge sharing and transfer
- Innovation pipeline development from research insights
- Due diligence for partnerships and acquisitions

### Consultants & Analysts
- Rapid domain expertise development
- Client research and background preparation
- Knowledge asset building across engagements
- Thought leadership content development

### Independent Researchers
- Self-directed learning and expertise building
- Connection identification across disciplines
- Research question development and refinement
- Publication and collaboration opportunity discovery

## Productivity Patterns

### Daily Reading (60-90 minutes)
- Morning focused reading session
- Immediate note integration while insights are fresh
- Queue management and prioritization
- Connection identification and linking

### Weekly Synthesis (2-3 hours)
- Pattern recognition across week's reading
- Knowledge network analysis and optimization
- Gap identification and reading planning
- Research direction evaluation and adjustment

### Monthly Review (4-6 hours)
- Domain landscape analysis using network visualization
- Research question evolution assessment
- Collaboration opportunity identification
- Knowledge base health monitoring and improvement

## Based On

This skill is inspired by the open-source [scholar](https://github.com/EESJGong/scholar) project by EESJGong (141 GitHub stars), which pioneered systematic academic reading workflows with networked knowledge building.

---

**Stop reading papers. Start building knowledge.**

Transform every paper into lasting intellectual assets that compound over time.