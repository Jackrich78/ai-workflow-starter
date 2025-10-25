# Architecture Decision: Specialist Sub-Agent Creation System

**Feature ID:** FEAT-003
**Decision Date:** 2025-10-25
**Status:** Proposed

## Context

The current agent system uses five fixed general-purpose sub-agents without ability to create domain-specific specialists. When projects use specialized technologies (Supabase, PydanticAI, FastAPI), agents repeatedly research the same topics every session, wasting time and context. We need systematic specialist creation that persists technology expertise per-project, integrates seamlessly with existing Explorer/Researcher workflows, completes in under 2 minutes, and uses Archon RAG → WebSearch → User cascade for knowledge gathering. This must work with or without Archon MCP, not break current workflows, and follow existing agent architecture patterns.

## Options Considered

### Option 1: Hybrid Template-Based Creation with Research Auto-Population

**Description:** Use TEMPLATE.md for instant structural scaffolding, then auto-populate specialist instructions via 90-second research using Archon RAG → WebSearch cascade. Narrow specialization by default (e.g., "Supabase Specialist" over "Database Specialist") with user option to broaden. Stateless specialists invoked via Task tool by Explorer and Researcher agents.

**Key Characteristics:**
- Two-phase creation: Static scaffolding (30s) + research-driven population (90s) = <2 min total
- Auto-populates name, description, tools, objectives, best practices from Archon/Web research
- Narrow scope default (library-specific) with user override to broaden
- Stateless design: each invocation independent, no knowledge accumulation (Phase 1)
- Gatekeeper pattern: Explorer/Researcher invoke specialists as subordinates via Task tool
- Library detection via regex pattern matching during Explorer Q&A with confirmation dialogue

**Example Implementation:**
```markdown
# Flow during /explore
User mentions "Supabase" → Explorer detects via regex
Explorer: "I noticed you mentioned Supabase. Create specialist?"
User: "Yes, narrow scope"
→ Load TEMPLATE.md (30s)
→ Query Archon: "Supabase tools patterns" (20s)
→ Query Web: "Supabase best practices 2025" (40s)
→ Populate specialist instructions (30s)
→ Write .claude/subagents/supabase-specialist.md
Explorer later invokes: @.claude/subagents/supabase-specialist.md via Task tool
```

### Option 2: Pure Static Template with Manual User Population

**Description:** Use TEMPLATE.md for instant scaffolding with minimal auto-population (only name/description). User manually edits specialist file to add tools, objectives, and best practices. Fastest creation (<10 seconds) but provides minimal immediate value.

**Key Characteristics:**
- Single-phase creation: Static scaffolding only
- Minimal auto-population (name from library name, generic description)
- User must manually populate tools list, primary objective, core responsibilities
- Same narrow specialization default and stateless design as Option 1
- Same gatekeeper invocation pattern via Task tool
- No research dependencies (works offline)

**Example Implementation:**
```markdown
# Flow during /explore
User mentions "Supabase" → Explorer detects
Explorer: "Create Supabase specialist?"
User: "Yes"
→ Load TEMPLATE.md (5s)
→ Substitute {{NAME}} = "supabase", {{DESCRIPTION}} = "Supabase database specialist" (2s)
→ Write .claude/subagents/supabase-specialist.md with TODOs for user to populate
User opens file, manually adds tools/objectives/patterns (5-20 minutes)
```

### Option 3: Broad Generalist Specialists with NLP-Based Library Detection

**Description:** Create broader specialists (e.g., "Backend Specialist" covers FastAPI, Django, Flask) using NLP entity extraction for library detection. Uses research auto-population like Option 1 but optimizes for fewer, more versatile specialists.

**Key Characteristics:**
- Broad scope default: "Database Specialist" handles Postgres, Supabase, MySQL collectively
- NLP-based entity extraction (spaCy or cloud API) for context-aware library detection
- Research-driven auto-population covering multiple related libraries
- Stateful design: specialists accumulate knowledge across invocations by updating instruction files
- Same gatekeeper invocation pattern via Task tool
- Higher initial creation time (2-3 minutes) due to broader research scope

**Example Implementation:**
```markdown
# Flow during /explore
User: "We'll use Supabase for database and FastAPI for backend"
Explorer uses spaCy NER: detects "Supabase" (database), "FastAPI" (backend framework)
Explorer: "Create Database Specialist (covers Postgres, Supabase, MySQL)?"
User: "Yes"
→ Research databases: Postgres, Supabase, MySQL patterns (120s)
→ Populate specialist with multi-library expertise
→ Write .claude/subagents/database-specialist.md
Specialist updates own file after each invocation with new findings
```

## Comparison Matrix

| Criteria | Option 1: Hybrid Template + Research | Option 2: Pure Static Template | Option 3: Broad Generalist + NLP |
|----------|----------|----------|----------|
| **Feasibility** | ✅ Easy to implement, uses existing patterns | ✅ Trivial implementation, no dependencies | ⚠️ Moderate complexity (NLP integration, state management) |
| **Performance** | ✅ <2 min creation meets requirement | ✅ <10s creation, fastest option | ❌ 2-3 min creation exceeds target |
| **Maintainability** | ✅ Auto-population reduces user maintenance | ❌ High maintenance burden on users | ⚠️ Stateful updates add complexity |
| **Cost** | ✅ Free (Archon local, WebSearch available) | ✅ Zero cost, no external dependencies | ⚠️ NLP APIs cost money or require local spaCy setup |
| **Complexity** | ⚠️ Two-phase creation, cascade logic | ✅ Minimal complexity, one-step creation | ❌ High complexity (NLP, state, broad scope) |
| **Community/Support** | ✅ Follows LangChain, AutoGen, PydanticAI patterns | ✅ Standard templating approach | ⚠️ Requires NLP library expertise |
| **Integration** | ✅ Seamless with existing Explorer/Researcher | ✅ No changes to existing agents required | ⚠️ Requires Explorer modification for NLP |

### Criteria Definitions

- **Feasibility:** Can we implement with current resources/skills/timeline?
- **Performance:** Meets <2 minute creation target?
- **Maintainability:** How easy to modify specialists and maintain knowledge quality?
- **Cost:** Financial cost (APIs, services, infrastructure)?
- **Complexity:** Implementation and operational complexity?
- **Community/Support:** Alignment with industry patterns and best practices?
- **Integration:** How well integrates with existing /explore workflow?

## Recommendation

**Chosen Approach:** Option 1 - Hybrid Template-Based Creation with Research Auto-Population

**Rationale:**

Option 1 delivers optimal balance of speed (<2 minutes), quality (comprehensive specialist instructions via research), and simplicity (stateless design, graceful Archon degradation). Research shows 91% of developers use AI for boilerplate generation, validating research-driven auto-population approach. Narrow specialization by default aligns with research finding that specialists outperform generalists in their domain, improving decision quality. Two-phase creation (static scaffolding + research population) meets time requirement while providing immediate value. Gatekeeper pattern fits existing Explorer/Researcher architecture without breaking changes. Stateless design avoids state management complexity while remaining reusable across features.

### Why Not Other Options?

**Option 2: Pure Static Template**
- Defeats purpose of specialist creation—users gain no immediate expertise advantage
- High maintenance burden requires users to manually research and populate tools/patterns (5-20 minutes)
- Minimal domain expertise in specialist reduces value proposition
- No competitive advantage over manually editing TEMPLATE.md directly

**Option 3: Broad Generalist + NLP**
- Research shows narrow specialists outperform broad generalists, contradicting specialization benefit
- 2-3 minute creation time exceeds <2 minute requirement
- NLP integration adds unnecessary complexity for explicit library mentions during Q&A
- Stateful design introduces state management complexity premature for Phase 1
- Higher cost (NLP APIs) or setup burden (local spaCy installation)

### Trade-offs Accepted

**Trade-off 1: Auto-population quality depends on Archon/Web content**
Mitigation: Graceful degradation to WebSearch if Archon lacks coverage. User can review and edit specialist before finalizing. Research-driven population still superior to empty static template.

**Trade-off 2: Narrow specialists increase specialist count**
Mitigation: Start with critical frameworks only (test case: Supabase). Create specialists on-demand during /explore, not preemptively. Documenter indexes all specialists for visibility.

**Trade-off 3: Stateless design requires redundant research across invocations**
Mitigation: Accept redundancy in Phase 1 for simplicity. Plan stateful enhancement (Phase 3) where specialists update own files with accumulated knowledge. Research cache (Archon RAG) reduces redundancy.

## Spike Plan

### Step 1: Validate Template Scaffolding
- **Action:** Load TEMPLATE.md, substitute placeholders for "Supabase Specialist" (name, description), write test file
- **Success Criteria:** File created at .claude/subagents/supabase-specialist.md with valid YAML frontmatter and markdown structure
- **Time Estimate:** 15 minutes

### Step 2: Test Archon RAG Auto-Population
- **Action:** Query Archon: "Supabase authentication patterns tools", extract relevant content, populate "Core Responsibilities" section
- **Success Criteria:** Specialist contains 3-5 Supabase-specific responsibilities with tool references, completed in <60 seconds
- **Time Estimate:** 30 minutes

### Step 3: Test WebSearch Fallback Population
- **Action:** Query WebSearch: "Supabase best practices 2025", extract findings, populate "Best Practices" or similar section
- **Success Criteria:** Specialist contains 3-5 best practices with source citations, completed in <60 seconds
- **Time Estimate:** 30 minutes

### Step 4: Validate Library Detection During Explorer Q&A
- **Action:** Simulate /explore conversation where user mentions "Supabase", implement regex detection for well-known libraries
- **Success Criteria:** Explorer detects "Supabase" mention, prompts user with specialist creation choice (narrow/broad/skip)
- **Time Estimate:** 45 minutes

### Step 5: Test Specialist Invocation via Task Tool
- **Action:** Create Researcher invocation calling @.claude/subagents/supabase-specialist.md, pass question "How to implement RLS in Supabase?"
- **Success Criteria:** Specialist returns answer using Archon RAG → WebSearch cascade, documents knowledge source
- **Time Estimate:** 30 minutes

**Total Spike Time:** 2.5 hours

## Implementation Notes

### Architecture Diagram
```
User runs /explore
  ↓
Explorer detects library mention ("Supabase") via regex
  ↓
Explorer prompts: "Create Supabase Specialist? (narrow/broad/skip)"
  ↓
User confirms → Specialist Creation Workflow
  ↓
[Phase 1: Static Scaffolding - 30s]
Load TEMPLATE.md → Substitute {{NAME}}, {{DESCRIPTION}}, {{LIBRARY}} placeholders
  ↓
[Phase 2: Research Auto-Population - 90s]
Query Archon RAG: "[Library] tools patterns best practices"
  ↓ (if insufficient)
Query WebSearch: "[Library] documentation getting started 2025"
  ↓
Populate sections: Primary Objective, Core Responsibilities, Tools Access, Examples
  ↓
Write .claude/subagents/[library-name]-specialist.md
  ↓
Explorer continues → Researcher invoked
  ↓
Researcher calls specialist via Task tool when domain question arises
  ↓
Specialist uses knowledge cascade:
  1. Check Archon RAG (if available)
  2. Fallback to WebSearch
  3. Fallback to User question
  ↓
Specialist returns findings to Researcher
  ↓
Researcher synthesizes specialist findings into research.md
```

### Key Components
- **Library Detector:** Regex pattern matching in Explorer agent for well-known frameworks (FastAPI, Supabase, PydanticAI, Django, React, etc.)
- **Template Scaffolder:** Loads TEMPLATE.md, performs string substitution for placeholders
- **Research Auto-Populator:** Orchestrates Archon RAG query → WebSearch query → content extraction → section population
- **Specialist File Writer:** Writes completed specialist to .claude/subagents/[library-name]-specialist.md with kebab-case naming
- **Knowledge Cascade Handler:** In specialist agent instructions, implements Archon → Web → User fallback logic

### Data Flow
1. User mentions library during Explorer Q&A ("We'll use Supabase for database")
2. Explorer regex matches "Supabase" against known library patterns
3. Explorer prompts user with specialist creation choice
4. User confirms "Yes, narrow scope" → Creation workflow begins
5. Load TEMPLATE.md, substitute {{NAME}} = "supabase", {{DESCRIPTION}} = "Supabase database specialist"
6. Query Archon: "Supabase authentication RLS patterns tools" → Extract tools list, patterns
7. Query WebSearch: "Supabase best practices 2025 documentation" → Extract best practices, URLs
8. Populate specialist sections with research findings
9. Write .claude/subagents/supabase-specialist.md
10. Later: Researcher invokes specialist via Task tool for Supabase questions
11. Specialist uses Archon RAG → WebSearch → User cascade to answer
12. Specialist returns findings to Researcher
13. Researcher includes findings in research.md

### Technical Dependencies
- **Archon MCP:** Optional, for RAG knowledge base queries (graceful degradation if unavailable)
- **WebSearch:** Required fallback for research auto-population
- **TEMPLATE.md:** Required base template at .claude/subagents/TEMPLATE.md
- **Task Tool:** Existing Claude Code tool for agent invocation
- **Library Regex Patterns:** Maintained list of well-known frameworks for detection

### Configuration Required
- **Library Detection Patterns:** Regex list in Explorer agent for common frameworks
- **Archon Sources:** Pre-crawled documentation for target libraries (Supabase, FastAPI, etc.)
- **Template Placeholders:** {{NAME}}, {{DESCRIPTION}}, {{LIBRARY}}, {{TOOLS}}, {{OBJECTIVE}} in TEMPLATE.md
- **Specialist Storage Path:** .claude/subagents/ directory (must exist or be created)

## Risks & Mitigation

### Risk 1: Auto-population quality varies with Archon/Web content availability
- **Impact:** High (poor specialist instructions reduce value)
- **Likelihood:** Medium (depends on Archon crawling and Web content quality)
- **Mitigation:** User reviews specialist before finalizing, can edit instructions. Fallback cascade (Archon → Web → User) ensures some content always available. Start with well-documented libraries (Supabase, FastAPI) for test cases.

### Risk 2: Library detection false positives/negatives
- **Impact:** Medium (creates wrong specialist or misses opportunity)
- **Likelihood:** Medium (regex may match "Paris" as framework or miss custom library names)
- **Mitigation:** Confirmation dialogue prevents false positive creation. User can skip and manually create later. Maintain curated list of known frameworks for high-precision matching.

### Risk 3: Specialist invocation overhead slows workflows
- **Impact:** Low (slightly longer research phase)
- **Likelihood:** Low (Task tool invocation is fast, specialists focus queries)
- **Mitigation:** Specialists are optional—Explorer/Researcher can skip invocation if answer is simple. Specialists use same tools (Read, Grep, WebSearch) so no additional latency vs. direct agent research.

## References

- [Research findings]: `docs/features/FEAT-003_specialist_subagent_creation/research.md`
- [PRD]: `docs/features/FEAT-003_specialist_subagent_creation/prd.md`
- [TEMPLATE.md]: `.claude/subagents/TEMPLATE.md`
- [LangChain Agent Patterns]: https://python.langchain.com/docs/get_started/introduction
- [PydanticAI Dependency Injection]: https://ai.pydantic.dev/
- [AutoGen Multi-Agent Patterns]: https://microsoft.github.io/autogen/

---

**Decision Status:** Proposed
**Next Steps:** Proceed to acceptance criteria and testing strategy
