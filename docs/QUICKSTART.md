# AI Workflow Starter - Quickstart Guide

**Last Updated:** 2025-10-24  
**Status:** Phase 1 Complete - Planning & Documentation

## What Is This?

A systematic framework for AI-assisted development using Claude Code. Instead of ad-hoc prompting, you get:

- **Structured agents** (planner, researcher, implementer, tester)
- **Slash commands** to trigger specific workflows (`/plan`, `/research`, `/build`)
- **Automated hooks** for formatting, testing, committing
- **Knowledge integration** via Archon MCP server
- **Full TDD pipeline** from planning through deployment

**Current Phase:** Phase 1 (Planning & Documentation) - **COMPLETE**  
**Next Phase:** Phase 2 (Implementation & Testing) - needs Archon integration

---

## Prerequisites

- **Claude Code CLI** installed and authenticated
- **Git** configured with GitHub access
- **Archon MCP Server** running locally (see below)
- **Docker** (for Archon)
- **Node.js 18+** or **Python 3.9+** or **Rust** (your project stack)

---

## Quick Setup (10 minutes)

### 1. Clone and Setup Repository

```bash
# Clone the repo
git clone https://github.com/Jackrich78/ai-workflow-starter.git
cd ai-workflow-starter

# Switch to development branch
git checkout archon  # or: git checkout -b dev
```

### 2. Setup Archon (Knowledge Base)

Archon is your project's knowledge management system. Claude Code uses it to access documentation.

```bash
# In a separate terminal/directory
cd ~/dev
git clone -b stable https://github.com/coleam00/archon.git
cd archon

# Configure environment
cp .env.example .env
# Edit .env and add:
# - SUPABASE_URL=https://your-project.supabase.co
# - SUPABASE_SERVICE_KEY=your-service-key-here
# - OPENAI_API_KEY=your-openai-key (or Gemini/Ollama)

# Setup database
# 1. Go to https://supabase.com/dashboard
# 2. Open SQL Editor
# 3. Copy/paste contents of migration/complete_setup.sql
# 4. Execute

# Start Archon
docker compose up --build -d

# Verify running
open http://localhost:3737
```

### 3. Connect Claude Code to Archon

**Get MCP Configuration:**
1. Open http://localhost:3737
2. Go to Settings → MCP Dashboard
3. Copy the configuration for Claude Code

**Add to Claude Code:**
```bash
# Edit your Claude Code config
# Location varies by OS:
# - macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
# - Linux: ~/.config/Claude/claude_desktop_config.json
# - Windows: %APPDATA%\Claude\claude_desktop_config.json

# Add the Archon MCP server config from the dashboard
```

**Restart Claude Code** to load the MCP connection.

### 4. Seed Archon Knowledge Base

Add your project documentation to Archon:

```bash
# Option 1: Crawl relevant documentation sites
# Go to http://localhost:3737 → Knowledge Base → Crawl Website
# Examples:
# - https://docs.claude.com (Claude API docs)
# - https://docs.anthropic.com (Anthropic docs)
# - Your tech stack docs (React, FastAPI, etc.)

# Option 2: Upload project docs
# Upload PDFs, markdown files, or text documents
```

---

## Using the System

### Current Capabilities (Phase 1)

**Available Commands:**
- `/plan <feature-name>` - Plan a new feature
- `/research <topic>` - Research a topic and document findings
- `/context` - Show current project context
- `/mistakes` - Review and learn from common mistakes

**Available Agents:**
- **Planner Agent** - Creates structured feature plans (PRD, architecture, tests)
- **Researcher Agent** - Conducts research and documents findings

### Example Workflow

```bash
# 1. Start Claude Code in your project
cd ~/dev/ai-workflow-starter
claude-code

# 2. Plan a new feature
> /plan user-authentication

# This will:
# - Create feature directory: docs/features/FEAT-002_user-authentication/
# - Generate: prd.md, architecture.md, testing.md, acceptance.md
# - Create test stubs based on requirements
# - Update CHANGELOG.md

# 3. Research if needed
> /research "best practices for JWT authentication in 2025"

# This will:
# - Use Archon to search existing knowledge
# - Search web if needed
# - Create research document in docs/archive/research-2025-10/
# - Update knowledge base

# 4. Review the plan
# - Check docs/features/FEAT-002_user-authentication/
# - Review all generated documents
# - Modify if needed

# 5. When satisfied, commit
git add .
git commit -m "feat: plan user authentication feature [FEAT-002]"
git push
```

---

## Project Structure

```
ai-workflow-starter/
├── docs/
│   ├── features/           # Feature-specific documentation
│   │   └── FEAT-XXX_name/
│   │       ├── prd.md      # Product requirements
│   │       ├── architecture.md
│   │       ├── testing.md
│   │       └── acceptance.md
│   ├── sop/                # Standard operating procedures
│   │   ├── git-workflow.md
│   │   ├── code-style.md
│   │   ├── testing-strategy.md
│   │   └── mistakes.md
│   ├── system/             # System-wide architecture
│   │   ├── architecture.md
│   │   ├── stack.md
│   │   ├── api.md
│   │   └── database.md
│   ├── archive/            # Research and historical docs
│   │   └── research-2025-10/
│   └── templates/          # Documentation templates
├── tests/                  # Test files (per feature)
├── AC.md                   # Acceptance criteria tracking
├── CHANGELOG.md            # Project changelog
└── README.md               # Project overview
```

---

## Key Concepts

### Feature-Driven Development

Every feature goes through:
1. **Planning** (`/plan`) - Create comprehensive documentation
2. **Research** (`/research`) - Investigate unknowns
3. **Implementation** (`/build`) - *Phase 2*
4. **Testing** (`/test`) - *Phase 2*
5. **Deployment** - *Phase 3*

### Documentation First

Before writing code:
- Write PRD (what and why)
- Design architecture (how)
- Define tests (acceptance criteria)
- Create test stubs (TDD setup)

### Context Engineering

Archon provides Claude Code with:
- Project-specific documentation
- Best practices from crawled sources
- Historical decisions and research
- Task context and requirements

---

## Phase Roadmap

### ✅ Phase 1: Planning & Documentation (COMPLETE)
- Planner and Researcher agents
- `/plan` and `/research` commands
- Documentation structure and templates
- Git workflow established

### 🚧 Phase 2: Implementation & Testing (NEXT)
**Agents:**
- Implementer (writes code following TDD)
- Tester (runs test suites, generates reports)

**Commands:**
- `/build [FEAT-ID]` - Implement feature
- `/test [FEAT-ID]` - Run tests
- `/commit [message]` - Automated git workflow

**Hooks:**
- PostToolUse - Auto-format, run tests, update docs
- Stop - Show git status, suggest commit

### 🔮 Phase 3: Automation & Profiles (FUTURE)
- Enhanced Archon RAG integration
- Stack-specific profiles (React, FastAPI, Rust, etc.)
- CI/CD templates
- Monitoring and metrics

See `docs/future-enhancements.md` for full roadmap.

---

## Maintaining Documentation

### When to Update Docs

**After every feature:**
- Update `CHANGELOG.md` with changes
- Add to `AC.md` if acceptance criteria met
- Archive research in `docs/archive/`
- Update system docs if architecture changed

**During development:**
- Keep feature docs accurate
- Document mistakes in `docs/sop/mistakes.md`
- Update templates if patterns emerge

### Doc Update Pattern

```bash
# 1. Make changes to feature
# 2. Update relevant docs
# 3. Commit with clear message

git add docs/features/FEAT-XXX/ CHANGELOG.md
git commit -m "docs: update FEAT-XXX architecture for API changes"
```

---

## Troubleshooting

### Archon Not Connecting
```bash
# Check Archon is running
curl http://localhost:8051/health

# Check MCP config is correct
cat ~/.config/Claude/claude_desktop_config.json

# Restart Claude Code after config changes
```

### Commands Not Working
- Ensure you're in project root directory
- Check that `.claude/` directory has command files
- Verify agent instructions are present in `.claude/subagents/`

### Knowledge Base Empty
- Crawl documentation sites in Archon UI
- Upload project-specific documents
- Verify embeddings are generated (check Archon logs)

---

## Getting Help

1. **Check Documentation**: Start with `docs/README.md`
2. **Review SOPs**: See `docs/sop/` for workflows
3. **Archon Issues**: https://github.com/coleam00/Archon/discussions
4. **Project Issues**: https://github.com/Jackrich78/ai-workflow-starter/issues

---

## Next Steps

1. **Try the example workflow** above
2. **Plan your first feature** with `/plan`
3. **Explore the documentation** structure
4. **Customize for your stack** (update templates, SOPs)
5. **Wait for Phase 2** to start building!

---

## Contributing

This is an open framework. To contribute:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-idea`)
3. Follow the `/plan` workflow for your contribution
4. Submit a pull request

---
