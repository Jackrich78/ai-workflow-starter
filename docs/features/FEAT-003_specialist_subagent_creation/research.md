# Research Findings: Specialist Sub-Agent Creation System

**Feature ID:** FEAT-003
**Research Date:** 2025-10-25
**Researcher:** Researcher Agent

## Research Questions

*Questions from PRD that this research addresses:*

1. What patterns exist for dynamic agent creation in AI workflow systems?
2. How should specialists handle library version specificity?
3. What is the optimal template population strategy?
4. How should library/framework detection work during Explorer Q&A?
5. Should specialists accumulate knowledge across invocations?
6. What scope granularity works best for specialists?

## Findings

### Topic 1: Dynamic Agent Creation Patterns in AI Systems

**Summary:** Modern AI frameworks (LangChain, AutoGen, PydanticAI) use modular, composable architectures for dynamic agent creation, emphasizing runtime instantiation, tool injection, and orchestrator-worker patterns.

**Details:**
- **LangChain/LangGraph** provides graph-based agent architecture where agents are nodes with dynamic communication paths, supporting many-to-many connections and stateful transitions based on context
- **Microsoft AutoGen v0.4** (January 2025) uses actor model with asynchronous message-passing, enabling modular self-contained agents that can be developed, tested, and deployed independently
- **PydanticAI** demonstrates dependency injection patterns for tools and agent configuration, allowing dynamic tool registration, override, and conditional preparation based on runtime context
- Common pattern: Template-based scaffolding with auto-population of agent properties (name, description, tools, system prompts)
- All frameworks support subordinate agent invocation through orchestrator-worker or multi-agent gatekeeper patterns

**Source:** LangChain, AutoGen, PydanticAI Documentation
**URL:** https://medium.com/@iamanraghuvanshi/agentic-ai-3-top-ai-agent-frameworks-in-2025-langchain-autogen-crewai-beyond-2fc3388e7dec
**Retrieved:** 2025-10-25 via WebSearch

**Source:** PydanticAI Documentation - Agent Tools and Dependency Injection
**URL:** https://ai.pydantic.dev/llms-full.txt
**Retrieved:** 2025-10-25 via Archon RAG

### Topic 2: Library/Framework Detection Mechanisms

**Summary:** Modern entity extraction uses NLP-based contextual understanding rather than simple keyword matching, with hybrid approaches combining pattern matching for structured data and ML models for context disambiguation.

**Details:**
- **Entity Extraction Tools:** SpaCy (open-source NLP with built-in NER), NLTK (text processing platform), Flair (deep learning NER from Zalando Research)
- **Cloud APIs:** Google Cloud Natural Language API, Amazon Comprehend, IBM Watson NLU provide pre-trained entity extraction
- **Keywords vs. Entity Extraction:** Keywords only match words, while entity extraction uses context (e.g., "Paris" as city vs. person vs. non-entity)
- **Hybrid Approach Recommended:** Pattern matching for structured entities (library names like "FastAPI", "Supabase") + NLP for context validation
- **Detection Strategy:** Regex patterns for well-known frameworks during Explorer Q&A, followed by confirmation dialogue to avoid false positives

**Source:** Named Entity Recognition Comprehensive Guide
**URL:** https://medium.com/@kanerika/named-entity-recognition-a-comprehensive-guide-to-nlps-key-technology-636a124eaa46
**Retrieved:** 2025-10-25 via WebSearch

### Topic 3: Template Population and Code Scaffolding Best Practices

**Summary:** AI-powered code generation in 2025 emphasizes template-based scaffolding with comprehensive auto-population, using tools like Cookiecutter, Copier, and PyScaffold for Python, with 91% of developers using AI for boilerplate generation.

**Details:**
- **Best Practice:** Static template structure (TEMPLATE.md) + research-driven comprehensive population for specialist content
- **Scaffolding Tools:** Cookiecutter (project templates), Copier (updateable templates), PyScaffold (Python best practices)
- **AI Code Generation:** GitHub Copilot, JetBrains AI, Tabnine used for boilerplate, test scaffolding, and template population
- **Population Strategy:** Two-phase approach: (1) Instant static scaffolding for structure, (2) Research-based auto-population for domain-specific content (tools, objectives, best practices)
- **Quality Guidelines:** Be clear and specific in prompts, validate AI-generated content, ensure consistency with existing agent patterns

**Source:** The Evolving Landscape of AI Code Generation Tools in 2025
**URL:** https://saptak.in/writing/2025/03/10/ai-code-generation-tools
**Retrieved:** 2025-10-25 via WebSearch

**Source:** 12 Scaffolding Tools to Supercharge Your Development Workflow
**URL:** https://www.resourcely.io/post/12-scaffolding-tools
**Retrieved:** 2025-10-25 via WebSearch

### Topic 4: Stateful vs. Stateless Agents - Knowledge Accumulation

**Summary:** Stateful agents maintain persistent memory and accumulate knowledge over time, delivering richer interactions and continuous learning, while stateless agents treat each interaction independently without memory of past behavior.

**Details:**
- **Stateless Agents:** Treat every interaction independently, no memory beyond training weights, completely restart each invocation
- **Stateful Agents:** Maintain persistent memory, accumulate domain knowledge, recognize patterns in user behavior, build knowledge over time
- **Knowledge Persistence:** Well-designed stateful agents can add facts to knowledge graphs, store embeddings for events, archive outdated data
- **Memory Management:** Long-term state enables agents to go beyond transient context windows, building knowledge persistently
- **Trade-offs:** Stateful requires state management infrastructure but delivers personalized experiences; stateless is simpler but requires redundant research
- **Recommendation for Specialists:** Hybrid approach - stateless invocation with optional knowledge persistence via specialist file updates (manual or automated based on confidence)

**Source:** Stateful Agents: The Missing Link in LLM Intelligence
**URL:** https://www.letta.com/blog/stateful-agents
**Retrieved:** 2025-10-25 via WebSearch

**Source:** Stateful vs Stateless AI Agents: Know Key Differences
**URL:** https://insights.daffodilsw.com/blog/stateful-vs-stateless-ai-agents-when-to-choose-each-pattern
**Retrieved:** 2025-10-25 via WebSearch

### Topic 5: Agent Specialization Scope (Broad vs. Narrow)

**Summary:** Narrow, specialized agents outperform generalists in their domain by reducing noise and focusing on key variables, with research showing narrower AI features are easier for users to understand and adopt.

**Details:**
- **Narrow AI (Weak AI):** Highly specialized, limited to specific tasks, outperforms humans in tightly specified domains (e.g., "Supabase Specialist")
- **Broad AI:** Handles diverse tasks but risks overfitting to wrong intent and selecting suboptimal tools in high-stakes situations
- **Scope Design Strategy:** Start small with well-defined, granular sub-tasks, reducing complexity while maintaining autonomy
- **User Adoption:** Narrower AI features are typically easier for new users to understand and adopt
- **Trade-offs:** Specialists outperform in their domain but lack generalization; generalists sacrifice precision for versatility
- **Recommendation:** Offer scope choice during creation (broad like "Database Specialist" vs. narrow like "Supabase Specialist"), with narrow as default for better initial performance

**Source:** Why AI Agents Should Be Specialists, Not Generalists
**URL:** https://www.kubiya.ai/blog/why-should-ai-agents-be-specialists-not-generalists-moe-in-practice
**Retrieved:** 2025-10-25 via WebSearch

**Source:** Scope in Generative AI Features
**URL:** https://www.nngroup.com/articles/scope-ai-features/
**Retrieved:** 2025-10-25 via WebSearch

### Topic 6: Library Version Specificity vs. Version-Agnostic Documentation

**Summary:** Technical API documentation is inherently version-specific, while conceptual guides and migration documentation can be version-agnostic, with best practice being to document "the latest" while providing access to past versions for users upgrading irregularly.

**Details:**
- **Version-Specific:** API docs must match software version to accurately reflect features, functions, and behaviors
- **Version-Agnostic:** Conceptual guides, design patterns, migration tips do not benefit from multiple versions
- **Best Practice:** Document "the latest" at primary location, offer past versions at predictable subfolders for wide audiences upgrading irregularly
- **Documentation-Code Coupling:** Docs must update with every code release for accuracy, but code should not release for every doc change
- **Recommendation for Specialists:** Version-agnostic approach initially (focus on concepts, patterns, best practices), with optional version pinning if user specifies version requirements

**Source:** Versioning Design Systems
**URL:** https://medium.com/eightshapes-llc/versioning-design-systems-48cceb5ace4d
**Retrieved:** 2025-10-25 via WebSearch

**Source:** How to cope with versioning of software documentation
**URL:** https://stackoverflow.com/questions/2356233/how-to-cope-with-versioning-of-software-documentation-and-the-software-itself
**Retrieved:** 2025-10-25 via WebSearch

### Topic 7: Subordinate Agent Invocation Patterns

**Summary:** AI workflow systems use orchestrator-worker, multi-agent gatekeeper, and sequential orchestration patterns for subordinate agent invocation, with hierarchical coordination and task delegation enabling specialization and resource optimization.

**Details:**
- **Orchestrator-Worker Pattern:** Central LLM breaks down tasks dynamically, delegates to workers, synthesizes results; flexible for unpredictable subtasks
- **Multi-Agent Gatekeeper Pattern:** Primary agent coordinates specialist subordinates, providing clear hierarchy and optimized resource use
- **Sequential Orchestration:** Step-by-step processing with deterministic agent invocation order, suitable for workflows with clear dependencies
- **Workflow Orchestration Agents:** Manage multistep tasks, delegate to subagents, maintain execution context, adapt based on intermediate results
- **Common Features:** All patterns support dynamic invocation, maintain context, specialize workers, synthesize outputs
- **Recommendation:** Use gatekeeper pattern (Explorer/Researcher as gatekeepers) with dynamic specialist invocation via Task tool

**Source:** AI Agent Orchestration Patterns - Azure Architecture
**URL:** https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns
**Retrieved:** 2025-10-25 via WebSearch

**Source:** AWS Prescriptive Guidance - Workflow Orchestration Agents
**URL:** https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/workflow-orchestration-agents.html
**Retrieved:** 2025-10-25 via WebSearch

## Recommendations

### Primary Recommendation: Hybrid Template-Based Creation with Narrow Specialization

**Rationale:**
- **Two-Phase Template Population:** Use TEMPLATE.md for instant structural scaffolding, then auto-populate domain-specific content (tools, objectives, best practices) via quick research (Archon RAG → WebSearch cascade)
- **Narrow Specialization by Default:** Default to narrow scope (e.g., "Supabase Specialist" over "Database Specialist") with user option to broaden, improving initial performance and user adoption
- **Stateless with Optional Persistence:** Specialists remain stateless for simplicity, with future enhancement for knowledge accumulation via specialist file updates
- **Version-Agnostic Initially:** Focus on concepts and patterns rather than version-specific APIs, allowing user to specify version requirements if needed
- **Gatekeeper Pattern:** Explorer and Researcher invoke specialists via Task tool, maintaining hierarchical orchestration

**Key Benefits:**
- **Speed:** Static scaffolding completes in <30 seconds, auto-population adds <90 seconds research time, meeting <2-minute creation goal
- **Quality:** Research-driven population ensures comprehensive specialist instructions following PydanticAI dependency injection and tool patterns
- **Flexibility:** Scope choice (broad/narrow) and version-agnostic approach accommodate diverse project needs
- **Simplicity:** Stateless design avoids state management complexity while maintaining reusability
- **Integration:** Gatekeeper pattern fits existing workflow without breaking Explorer or Researcher agents

**Considerations:**
- Auto-population quality depends on Archon RAG coverage; graceful degradation to WebSearch ensures functionality
- Narrow specialists may require multiple specialists for complex domains (e.g., separate Postgres and Supabase specialists)
- Future stateful enhancement needed for true knowledge accumulation across invocations

### Alternative Approaches Considered

#### Alternative 1: Pure Static Template (No Auto-Population)
- **Pros:** Fastest creation (<10 seconds), simplest implementation, no research dependencies
- **Cons:** Minimal domain expertise in specialist, requires user to manually populate tools and objectives, low initial value
- **Why not chosen:** Defeats purpose of specialist creation—users gain no immediate expertise advantage

#### Alternative 2: Broad Generalist Specialists (e.g., "Backend Specialist")
- **Pros:** Fewer specialists to manage, handles wider range of questions, simpler user choice
- **Cons:** Lower performance in specific domains, decision quality drops with overly broad scope, loses specialization benefit
- **Why not chosen:** Research shows narrow specialists outperform generalists; defeats purpose of specialization

## Trade-offs

### Performance vs. Complexity
**Narrow specialists** deliver superior domain performance but increase specialist count and management complexity. Mitigation: Start with critical frameworks only (test case: Database Specialist for Postgres/Supabase), expand on-demand.

### Static vs. Dynamic Population
**Static templates** are instant but provide minimal value. **Dynamic auto-population** requires 90 seconds research but delivers comprehensive specialist instructions. Choice: Dynamic for better quality, with static fallback if research fails.

### Stateless vs. Stateful Knowledge
**Stateless specialists** are simpler but require redundant research across invocations. **Stateful specialists** accumulate knowledge but add state management complexity. Choice: Stateless Phase 1, stateful enhancement Phase 3.

### Version-Specific vs. Version-Agnostic
**Version-specific** specialists are precise but require updates with library changes. **Version-agnostic** specialists focus on patterns but may miss version-specific features. Choice: Version-agnostic initially with user override option.

## Resources

### Official Documentation
- [LangChain Documentation](https://python.langchain.com/docs/get_started/introduction)
- [Microsoft AutoGen](https://microsoft.github.io/autogen/)
- [PydanticAI Documentation](https://ai.pydantic.dev/)
- [Semantic Versioning](https://semver.org/)

### Technical Articles
- [Top AI Agent Frameworks in 2025](https://medium.com/@iamanraghuvanshi/agentic-ai-3-top-ai-agent-frameworks-in-2025-langchain-autogen-crewai-beyond-2fc3388e7dec) (Aman Raghuvanshi, 2025-10-24)
- [AI Code Generation in 2025](https://saptak.in/writing/2025/03/10/ai-code-generation-tools) (Saptak, 2025-03-10)
- [Stateful Agents: The Missing Link](https://www.letta.com/blog/stateful-agents) (Letta, 2025)
- [Why AI Agents Should Be Specialists](https://www.kubiya.ai/blog/why-should-ai-agents-be-specialists-not-generalists-moe-in-practice) (Kubiya, 2025)

### Design Patterns
- [AI Agent Orchestration Patterns - Azure](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)
- [AWS Agentic AI Patterns](https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/introduction.html)

### NLP and Entity Extraction
- [Named Entity Recognition Guide](https://medium.com/@kanerika/named-entity-recognition-a-comprehensive-guide-to-nlps-key-technology-636a124eaa46) (Kanerika, 2025)
- [6 Best NER APIs](https://www.assemblyai.com/blog/6-best-named-entity-recognition-apis-entity-detection) (AssemblyAI)

### Scaffolding Tools
- [12 Scaffolding Tools](https://www.resourcely.io/post/12-scaffolding-tools) (Resourcely, 2025)
- [PyScaffold GitHub](https://github.com/pyscaffold/pyscaffold)

## Archon Status

### Knowledge Base Queries
*Archon MCP was available and used for research:*
- ✅ **PydanticAI Documentation** (source_id: c0e629a894699314): Found comprehensive agent patterns, dependency injection examples, tool registration code
- ✅ **FastAPI Documentation** (source_id: c889b62860c33a44 and 0c899ab3cf0ce5da): Available for future specialist creation testing
- ✅ **Supabase Documentation** (source_id: 9c5f534e51ee9237): Available for test case validation
- ⚠️ **LangChain/AutoGen Documentation**: Not in Archon, used WebSearch for framework comparison
- ⚠️ **NLP/NER Libraries**: Not in Archon, used WebSearch for entity extraction research
- ⚠️ **AI Orchestration Patterns**: Not in Archon, used WebSearch for Microsoft Azure and AWS guides

### Recommendations for Archon

1. **LangChain Documentation**
   - **Why:** Leading AI agent framework in 2025, critical for understanding multi-agent patterns and LangGraph orchestration
   - **URLs to crawl:**
     - https://python.langchain.com/docs/get_started/introduction
     - https://langchain-ai.github.io/langgraph/
   - **Benefit:** Enables faster research on agent creation patterns, workflow orchestration, and subordinate agent invocation for future features

2. **Microsoft AutoGen Documentation**
   - **Why:** AutoGen v0.4 (January 2025) represents cutting-edge multi-agent collaboration patterns with actor model and cross-language support
   - **URLs to crawl:**
     - https://microsoft.github.io/autogen/
     - https://github.com/microsoft/autogen
   - **Benefit:** Provides best practices for modular agent design, asynchronous messaging, and dynamic agent creation

3. **spaCy NLP Documentation**
   - **Why:** Open-source NLP library widely used for entity extraction, NER, and text processing—critical for library detection mechanisms
   - **URLs to crawl:**
     - https://spacy.io/usage
     - https://spacy.io/api
   - **Benefit:** Supports research on context-aware library detection beyond simple keyword matching

4. **Cookiecutter/Copier Documentation**
   - **Why:** Industry-standard Python project templating tools relevant to template population strategies
   - **URLs to crawl:**
     - https://cookiecutter.readthedocs.io/
     - https://copier.readthedocs.io/
   - **Benefit:** Informs best practices for template-based code generation and updateable scaffolding

## Answers to Open Questions

### Question 1: What patterns exist for dynamic agent creation in AI workflow systems?
**Answer:** Modern AI frameworks use modular, composable architectures with three primary patterns: (1) LangChain's graph-based agents with many-to-many connections and stateful transitions, (2) AutoGen's actor model with asynchronous message-passing and self-contained agents, (3) PydanticAI's dependency injection with dynamic tool registration and conditional preparation. All frameworks support template-based scaffolding with runtime instantiation and orchestrator-worker subordinate invocation patterns.

**Confidence:** High
**Source:** LangChain, AutoGen, PydanticAI documentation via Archon RAG and WebSearch (Topic 1)

### Question 2: How should specialists handle library version specificity?
**Answer:** Use version-agnostic approach initially, focusing on concepts, patterns, and best practices that remain stable across versions. Allow user to specify version requirements if needed, then optionally pin specialist to specific version. This mirrors industry best practice: technical API docs are version-specific, but conceptual guides are version-agnostic. Document "the latest" by default while supporting past versions for users upgrading irregularly.

**Confidence:** High
**Source:** Software documentation versioning best practices via WebSearch (Topic 6)

### Question 3: What is the optimal template population strategy?
**Answer:** Hybrid two-phase approach: (1) Instant static scaffolding using TEMPLATE.md for structure (name, tools, sections), (2) Research-driven comprehensive auto-population for domain-specific content (objectives, best practices, tool usage patterns) using Archon RAG → WebSearch cascade. This completes in <2 minutes (30 seconds scaffolding + 90 seconds research), balances speed with quality, and aligns with industry practice where 91% of developers use AI for boilerplate generation.

**Confidence:** High
**Source:** AI code generation and scaffolding tools research via WebSearch (Topic 3)

### Question 4: How should library/framework detection work during Explorer Q&A?
**Answer:** Use hybrid approach combining regex pattern matching for well-known framework names (FastAPI, Supabase, PydanticAI, etc.) with confirmation dialogue to avoid false positives. Simple keyword matching suffices for explicit library mentions during Q&A, as context is clear. NLP-based entity extraction is overkill for this use case but could be future enhancement for detecting implicit framework references in user responses.

**Confidence:** Medium
**Source:** NER and entity extraction research via WebSearch (Topic 2)

### Question 5: Should specialists accumulate knowledge across invocations?
**Answer:** Start with stateless design (Phase 1) for simplicity, avoiding state management complexity. Each specialist invocation is independent, requiring fresh research but remaining predictable. Plan for stateful enhancement (Phase 3) where specialists can update their own instruction files with accumulated knowledge (add findings to "Core Responsibilities" section, expand "Common Patterns" from invocations). This aligns with research showing stateful agents deliver richer interactions but require infrastructure investment.

**Confidence:** High
**Source:** Stateful vs. stateless agents research via WebSearch (Topic 4)

### Question 6: What scope granularity works best for specialists?
**Answer:** Narrow specialization outperforms broad generalists. Default to narrow scope (e.g., "Supabase Specialist" over "Database Specialist") with user option to broaden during creation. Research shows narrow AI features are easier for users to understand and adopt, specialized agents reduce noise by focusing on key variables, and decision quality drops in high-stakes situations with overly broad scope. Trade-off: more specialists to manage, but superior domain performance justifies complexity.

**Confidence:** High
**Source:** Agent specialization scope research via WebSearch (Topic 5)

## Next Steps

1. **Architecture Planning:** Evaluate implementation options for template population (static vs. research-driven), library detection mechanisms (regex vs. NLP), and specialist invocation patterns (Task tool vs. direct call)

2. **Technical Spike:**
   - Test TEMPLATE.md-based scaffolding with example "Supabase Specialist" creation
   - Validate Archon RAG → WebSearch cascade for auto-population (target: 90-second research time)
   - Prototype Explorer integration for library detection during Q&A
   - Test specialist invocation by Researcher agent via Task tool

3. **Proceed to Planning:** Run `/plan FEAT-003` with these research findings to create architecture, acceptance criteria, testing strategy, and test stubs

---

**Research Complete:** ✅
**Ready for Planning:** Yes
