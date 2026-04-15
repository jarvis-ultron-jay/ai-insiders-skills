# 🧠 AI Insiders Skills

**The AI skill warehouse. Built by practitioners, for practitioners.**

OpenClaw-compatible skills for anyone serious about integrating AI into their work and life.

## What Are Skills?

Skills are executable SOPs for AI agents. Each skill contains:

- **SKILL.md** — The core instruction file. Tells the agent what the skill does, when to use it, and exactly how to execute it step by step.
- **references/** — Supporting documents, templates, API specs, examples.
- **scripts/** — Executable scripts the agent can run directly.

When an agent receives a request, it matches against skill descriptions, picks the right one, reads the SKILL.md, and follows it autonomously.

## Categories

| Category | Skills |
|----------|--------|
| 🎨 Content & Copy | humanizer, word-docx, excel-xlsx, nano-pdf |
| 📱 Social Media | social-media-scheduler, social-intelligence |
| 🔍 Research & News | news-summary, blogwatcher, stock-analysis, youtube-watcher |
| 🤖 Agent Intelligence | self-improving, proactive-agent-lite, ontology |
| 🛠️ Operations | automation-workflows, obsidian, desktop-control, browser-use |
| 📧 Communications | imap-smtp-email, pollyreach, slack |
| 📊 Analytics | admapix, metricool |
| 🌤️ Utilities | weather, video-frames, github, gog, skill-creator |

## Installation

Install any skill into your OpenClaw workspace:

```bash
openclaw skills install <skill-name>
```

Or clone this repo and copy individual skill folders into your `~/.openclaw/workspace/skills/` directory.

## Structure

```
wild-ai-skills/
├── README.md
├── skill-name/
│   ├── SKILL.md          # Core instruction file
│   ├── meta.json         # Name, description, category, icon
│   ├── references/       # Supporting documents
│   └── scripts/          # Executable scripts
└── ...
```

## Contributing

Skills are added as they're built or curated. Each skill is vetted for security and quality before inclusion.

---

*Built by [AI Insiders](https://ai-insiders.ai) • Powered by [OpenClaw](https://openclaw.ai)*
