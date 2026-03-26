# Discover

## Intent

Deeply investigate a product, solution, technology, or domain to understand what it offers, how it works, and what is useful. The output is a structured summary of findings — not a plan, not a ticket, not an implementation. Discovery exists to build informed understanding before committing to any course of action.

Discovery may be broad ("what does this product do and how could we use it?") or narrow ("does this API support webhook filtering by event type?"). The scope of investigation should match the scope of the request. Minimal guidance produces exploratory discovery; specific guidance produces targeted discovery.

## When to Discover

- Evaluating a product, service, or tool for potential adoption or integration
- Investigating an API, SDK, or platform's capabilities before planning work against it
- Understanding a technology domain well enough to make informed decisions
- Answering a specific technical feasibility question that requires research
- Any time the question is "what is possible here?" rather than "what should we build?"

Discovery is not planning. It informs planning. A discovery document may lead to tickets, plans, or a decision to do nothing — all are valid outcomes.

## Document Storage & Naming

Discovery documents live inside a ticket folder when one exists, or in their own folder under `docs/pending/` when discovery is standalone:

- `docs/pending/07-investigate-pipeline-jobs/discover.md`
- `docs/pending/PROJ-123-investigate-pipeline-jobs/discover.md`
- `docs/pending/03-evaluate-monitoring-tools/discover.md`

The presence of `discover.md` in a folder indicates that discovery has been conducted. A folder may contain both `discover.md` and `ticket.md` when discovery was prompted by a ticket, or `discover.md` alone when discovery preceded any ticket.

## Procedure

### 1. Understand the Request

Clarify what is being investigated and why. Establish:

- **Subject** — the product, API, technology, or domain under investigation
- **Motivation** — what decision or work this discovery is meant to inform
- **Scope** — broad exploration or specific questions to answer

If the request is vague, ask the human for clarification before beginning deep investigation. A few minutes of scoping saves hours of unfocused research.

### 2. Investigate

Research the subject thoroughly within the established scope. This may include:

- Official documentation, API references, and specifications
- Available endpoints, methods, data formats, and authentication mechanisms
- Capabilities, features, and limitations
- Pricing, licensing, and access constraints
- Integration patterns and existing tooling (SDKs, CLI tools, libraries)
- Community resources, known issues, and maturity indicators

Follow leads. Discovery is not a checklist — pursue what is relevant and interesting as the investigation unfolds. Adjust depth based on what you find: spend more time on areas that are promising or complex, less on areas that are straightforward or clearly irrelevant.

### 3. Assess Usefulness

For each significant capability or finding, evaluate:

- Is this relevant to the motivation established in step 1?
- How could this be used? What would it enable?
- What are the constraints, limitations, or risks?
- What is missing or unclear that would need further investigation?

Separate what is confirmed from what is inferred or uncertain. Flag assumptions explicitly.

### 4. Write the Discovery Document

Produce `discover.md` with the findings. Structure the document around what was discovered, not around the process of discovering it. The document should be useful to someone who reads it without having participated in the investigation.

## Required Content

Every discovery document must include:

- **Subject** — what was investigated, including version or date context where relevant
- **Motivation** — why this investigation was conducted and what it is meant to inform
- **Summary** — a concise overview of key findings (a few sentences or short bullets that convey the essential picture)
- **Findings** — detailed account of what was discovered, organized by topic or capability area. Include specifics: endpoints, features, data formats, constraints, authentication requirements — whatever is relevant to the subject and scope
- **Assessment** — evaluation of usefulness, feasibility, risks, and gaps. What is promising, what is problematic, what remains unknown

## Optional Content

Include when relevant, omit when not:

- **Recommendations** — suggested next steps based on the findings (create a ticket, proceed to planning, investigate further, abandon)
- **Open Questions** — specific unknowns that could not be resolved during discovery and may need follow-up
- **References** — links to documentation, API references, or other sources consulted

## Guidance

- Discovery documents are reference material. Write them to be read later by someone (human or AI) who needs to understand the subject without repeating the investigation.
- Be specific. "The API supports filtering" is less useful than "The `/events` endpoint accepts a `type` query parameter that filters by event category; supported values are listed at [reference]."
- Separate facts from opinions. State what you found, then state what you think about it — clearly distinguished.
- Scope controls depth. A request to "look into X" warrants broad exploration. A request to "find out if X supports Y" warrants a focused answer. Match the document's depth to the request's scope.
- Do not pad discovery documents with background information the reader already knows. Focus on what was learned, not what was already understood.
- If discovery reveals enough information to act on, say so. If it reveals that more investigation is needed, say that too. The document should make the natural next step obvious.
