# Architecture Decision: Slack Block Kit Claude Skill

**Feature ID:** FEAT-004
**Created:** 2025-11-20
**Status:** Planning
**Decision Maker:** Planner Agent

## Context

We need to build a Claude Skill that generates valid Slack Block Kit JSON for three surfaces (Messages, Modals, Home Tabs) covering all 13 official block types. The Skill must support both N8N Jinja2 and Python variable templating, remain under 500 lines following Claude Skills best practices, and produce JSON that validates in Block Kit Builder.

**Key Constraints:**
- 500-line limit for main Skill file
- All 13 official Slack Block Kit block types must be covered
- Support for N8N (Jinja2) and Python variable templating
- Valid JSON output that passes Block Kit Builder validation
- Three surfaces with different block limits (Message: 50, Modal: 100, Home Tab: 100)
- Single-step modals only (v1 scope)

## Architecture Options

### Option 1: Progressive Disclosure with Supporting Docs (RECOMMENDED)

**Description:**
Main SKILL.md stays under 500 lines as the interaction interface. Supporting documentation files provide progressive disclosure of reference material, examples, and edge cases. Claude loads supporting docs on-demand when users ask specific questions.

**Structure:**
- `/skills/blockkit/SKILL.md` (450-500 lines) - Core interaction logic, overview, quick examples
- `/skills/blockkit/docs/blocks-reference.md` - All 15 block types with syntax
- `/skills/blockkit/docs/elements-guide.md` - Interactive elements (buttons, inputs, etc.)
- `/skills/blockkit/docs/composition-objects.md` - Text objects, option groups, etc.
- `/skills/blockkit/docs/surfaces.md` - Messages, Modals, Home Tabs specifications
- `/skills/blockkit/docs/variables-and-templating.md` - N8N Jinja2 and Python variable handling
- `/skills/blockkit/docs/examples.md` - Real-world use cases copied from Block Kit Builder
- `/skills/blockkit/docs/best-practices.md` - Validation, accessibility, common patterns

**Pros:**
- Aligns with Claude Skills best practice of progressive disclosure
- Main Skill stays focused and under 500 lines
- Easy to maintain and update individual reference sections
- Supports multi-model testing (different models load different levels of detail)
- Clear separation of concerns (interaction vs reference)

**Cons:**
- Requires multiple file structure (7-8 files total)
- Claude must read additional files on-demand (minimal latency)
- More complex initial setup

**Cost:** Development: 12-16 hours, Maintenance: Low (modular updates)

### Option 2: Monolithic Skill with Embedded Examples

**Description:**
Single SKILL.md file attempting to pack all information, examples, and guidance into 500 lines through heavy compression and minimal examples.

**Structure:**
- `/skills/blockkit/SKILL.md` (500 lines) - Everything compressed

**Pros:**
- Single file simplicity
- No additional file loading needed
- Faster initial response (all context loaded)

**Cons:**
- Violates 500-line best practice or requires extreme compression
- Difficult to maintain comprehensive coverage of 15 block types
- Limited space for examples (quality vs quantity trade-off)
- Hard to update without affecting entire Skill
- Poor separation of concerns
- Less suitable for multi-model testing (all-or-nothing context)

**Cost:** Development: 8-10 hours, Maintenance: High (full-file edits)

### Option 3: Skill + External Block Kit Builder Reference

**Description:**
Minimal Skill that teaches Claude to use Block Kit Builder documentation directly via web search or API, generating JSON by referencing live Slack docs.

**Structure:**
- `/skills/blockkit/SKILL.md` (200-300 lines) - Basic patterns and delegation to external docs

**Pros:**
- Minimal file size
- Always up-to-date with Slack changes (references live docs)
- Simplest to maintain

**Cons:**
- Requires external dependencies (web search, API access)
- Unpredictable quality (external docs change format)
- Slower response times (web requests)
- No guaranteed offline capability
- No control over example quality
- Violates "no external dependencies" requirement

**Cost:** Development: 6-8 hours, Maintenance: Very Low (mostly delegation)

## Comparison Matrix

| Criteria | Option 1: Progressive Disclosure | Option 2: Monolithic | Option 3: External Reference |
|----------|----------------------------------|----------------------|------------------------------|
| **Feasibility** | High - Proven Claude Skills pattern | Medium - Compression challenges | Low - External dependency requirement |
| **Performance** | High - On-demand loading | High - Single load | Low - Network latency |
| **Maintainability** | High - Modular updates | Low - Full-file edits | High - Minimal local code |
| **Cost** | Medium - Initial setup time | Low - Single file | Very Low - Delegation |
| **Complexity** | Medium - Multi-file orchestration | Low - Single file | High - External integration |
| **Community/Support** | High - Aligns with Claude best practices | Medium - Standard approach | Low - Custom integration |
| **Integration** | High - Self-contained, no external deps | High - Self-contained | Low - Requires external APIs |

## Recommendation

**Choose Option 1: Progressive Disclosure with Supporting Docs**

**Rationale:**
1. **Best Practice Alignment:** Follows Claude Skills documentation guidance for progressive disclosure
2. **Quality Over Compression:** Allows comprehensive coverage of all 15 block types without sacrificing example quality
3. **Maintainability:** Modular structure makes updates and additions straightforward
4. **No External Dependencies:** Fully self-contained, meeting requirements
5. **Multi-Model Friendly:** Different Claude models can load appropriate detail levels
6. **Professional Standards:** Production-quality approach suitable for long-term use

**Trade-offs Accepted:**
- More files to manage (7-8 vs 1)
- Slightly more complex initial setup
- On-demand file loading (negligible latency)

**Trade-offs Avoided:**
- Compressed examples that sacrifice clarity
- Maintenance burden of monolithic file
- External dependency risks
- Quality inconsistency

## Implementation Spike Plan

**Objective:** Validate progressive disclosure architecture with real Block Kit JSON generation

### Step 1: Create Main Skill Shell (2 hours)
- Create `/skills/blockkit/SKILL.md` with core structure (200 lines)
- Define interaction patterns and supporting doc references
- Include 3 quick examples (section block, button, modal)
- Test with Haiku, Sonnet, Opus models

**Validation:** All three models can generate basic valid JSON

### Step 2: Build Blocks Reference (3 hours)
- Create `/skills/blockkit/docs/blocks-reference.md`
- Document all 15 block types with syntax and examples
- Copy real examples from Block Kit Builder for accuracy
- Test Skill's ability to reference and use blocks

**Validation:** Generate JSON using 5 different block types, validate in Block Kit Builder

### Step 3: Add Variable Templating Guide (2 hours)
- Create `/skills/blockkit/docs/variables-and-templating.md`
- Document N8N Jinja2 syntax: `{{$node.data.field}}`
- Document Python approaches: f-strings, .format(), Jinja2 library
- Create examples showing placeholder → substitution flow

**Validation:** Generate JSON with N8N variables, manually substitute in N8N test workflow

### Step 4: Build Examples Library (3 hours)
- Create `/skills/blockkit/docs/examples.md`
- Add 10 real-world use cases: approval workflow, notification, task dashboard, etc.
- Manually copy and validate each from Block Kit Builder
- Cross-reference with blocks-reference.md

**Validation:** All 10 examples render correctly in Block Kit Builder, cover 12+ block types

### Step 5: Integration Testing (2 hours)
- Test complete Skill with complex multi-block scenarios
- Validate modal structure (input blocks, submit/cancel buttons)
- Test home tab dashboard generation
- Verify 50-block message and 100-block modal limits are documented
- Multi-model testing: Haiku (speed), Sonnet (balance), Opus (complexity)

**Validation:** 100% valid JSON output across 15 test scenarios, all models succeed

**Total Spike Duration:** 12 hours over 2-3 days

**Success Criteria:**
- Main SKILL.md under 500 lines
- All 15 block types covered
- Valid JSON output (100% pass rate in Block Kit Builder)
- N8N and Python variable guidance clear
- Multi-model testing successful
- No external dependencies

## Next Steps

1. **Approve Architecture:** Review and approve progressive disclosure approach
2. **Execute Spike Plan:** Follow 5-step validation plan
3. **Build Supporting Docs:** Create remaining reference files (elements, surfaces, best practices)
4. **Validate Acceptance Criteria:** Ensure all AC-FEAT-004-### criteria met
5. **Prepare for Phase 2:** Document any findings or adjustments from spike

## References

- Claude Skills Best Practices (Progressive Disclosure, 500-line limit)
- Slack Block Kit Builder: https://api.slack.com/tools/block-kit-builder
- Slack Block Kit Documentation: https://api.slack.com/block-kit
- N8N Jinja2 Templating: https://docs.n8n.io/code-examples/expressions/
- Related: `prd.md`, `research.md`, `acceptance.md`, `testing.md`
