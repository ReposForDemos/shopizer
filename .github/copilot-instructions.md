---
applyTo: '**'
---
General instructions for correct usage of CAST Imaging tools (tool names may reach you with a
prefix such as `bb7_`).

# Important information

CAST Imaging is a way of obtaining software intelligence: architectural information key to
understanding applications. It answers questions about dependencies, packages, caller/callee
relationships, transactions and data calls between user inputs and databases, quality findings,
and what the code actually means.

When asked questions related to software, technology, applications, or projects, reference the
Imaging tools. When the Structural pair is available, begin by identifying the application with
the `applications` function. Even if a repository is currently open, run it anyway — naming
conventions may differ between the repo and Imaging.

Do not ask the user "do you want me to use the tools" or "would you like me to proceed" — act.
Never recommend that the *user* run the tools; only you can call them.

If the user has no application open in the current repository, they may be asking a portfolio-level
question spanning multiple applications. Confirm that with them.

---

# How these tools work: syntax first, then execute

With the installer's default `intents` profiles and all three backends enabled, this server exposes
**six tools**, in three pairs. Each pair covers one kind of question:

| Pair | Discover with | Execute with |
|---|---|---|
| Structural | `get_structural_search_function_syntax` | `run_structural_search_function` |
| Semantic search | `get_semantic_search_function_syntax` | `run_semantic_search_function` |
| Text2Graph | `get_text2graph_function_syntax` | `run_text2graph_function` |

Everything else in this document — `applications`, `object_details`, `get_relevant_code_context`
and so on — is the name of a **function you pass to a `run_*` tool**, not a tool you can call
directly on the default surface. There is no directly advertised `applications` tool in that
profile; it is an `applications` function executed through `run_structural_search_function`.

An administrator can disable a backend or switch Structural or Semantic Search to its `full`
profile. Always trust the tools currently advertised by the connected server: if a pair is absent,
do not call it; if individual functions are advertised directly, call those tools directly instead
of inventing a missing syntax/execute pair.

**When the selected domain advertises a syntax/execute pair, use this workflow:**

1. Call the `get_*_syntax` tool for the functions you intend to use. It returns their exact
   parameters, defaults, and examples. Call it with an empty `function_names` list to see every
   function that pair offers.
2. Call the matching `run_*` tool with the function name and its arguments.

Never guess a function's parameters, and never pass a function name to the wrong `run_*` tool.
On the paired surface, never pass one of the six tool names as a `function_name`. Because the
syntax tool is the source of truth for parameters, prefer reading it over assuming — function
signatures change between releases, this document does not track them.

---

# Choosing the right pair

Route on the *shape* of the question, not on keywords:

- **Structural** — a known ID, an exact name, a filter pattern, graph topology, a metric, or any
  write action. Applications, objects, transactions, data graphs, source files, callers and
  callees, architecture, dependencies, quality findings, tags, views, post-its.
- **Semantic search** — meaning, purpose, behavior, business concepts, workflows, business rules,
  or relevant code/document context when the answer is not an exact structural fact.
- **Text2Graph** — last resort. Schema-aware, multi-hop graph exploration and custom read-only
  Cypher, used only when no purpose-built structural or semantic function fits. Always check
  the other two pairs first; reaching for raw Cypher without doing so is an anti-pattern.

When talking to the user, call the second capability **semantic search**. Users may say
"GraphRAG", "RAG", "graph RAG", "knowledge graph search" or "AI search" — they all mean semantic
search, and you should answer using that term.

Structural and semantic search complement each other. A good pattern for an unfamiliar codebase is
to locate the relevant code semantically, then pivot to structural functions for exact
dependencies, callers, and transactions.

Tie-breakers:
- "Where is order cancellation implemented?" → Semantic search.
- "Explain how payment processing works" → Semantic search.
- "Which transactions use this object?" → Structural.
- "Explain this identified transaction using its call graph" → Structural.
- "Which classes have the most incoming CALL relationships?" → Text2Graph, after confirming no
  purpose-built Structural or Semantic function answers it.

---

# The two major use cases

## 1) Exploration

Any general question about a whole application or portfolio: overall architecture, the main
sections of a system, technologies in use, quality posture.

Common questions:
- How do I migrate this technology?
- What are the current safety issues in my app? Which ones matter most?
- Summarize my application and its technologies.
- What are the main sections of my application?

Useful structural functions: `applications`, `stats`, `architectural_graph`,
`architectural_graph_focus`, `quality_insights`, `packages`, `object_profiles`,
`transaction_profiles`.

For exploration, avoid dropping into source code too early. Build the high-level picture from
Imaging first.

## 2) Impact analysis

Understanding the effects and dependencies of a change to specific objects or components. Perform
impact analysis before editing code, to gain context and avoid mistakes, and again after editing to
identify affected areas and tests worth running. Do not ask permission to do it — do it when it is
appropriate.

Useful structural functions: `object_details`, `transactions_using_object`,
`data_graphs_involving_object`, `inter_applications_dependencies`, `objects_relationships`,
`pathfinder_hierarchy_details`.

Identify the object first, then use those functions to trace the dependency and impact chains.
Consider which other code, databases, and technologies are affected — not just the direct callers.

### Edit process

**Before** the modification you suggest: gather context to design it well, minimizing or avoiding
breaking changes unless the user says otherwise.

**After** the modification you suggest:
- at minimum, tell the user which existing dependencies need care and which transactions should be
  tested;
- ideally, suggest follow-up modifications to repair breaking changes — recursively through the
  call graph if needed — and to update and run the affected tests.

When working with code elements:
- `quality_insight_violations` — find objects exhibiting a specific quality issue.
- `object_details` — on the object to change, and again on each dependent object.
- `transactions_using_object` — on every updated object; consolidate the transactions to retest.
- `data_graphs_involving_object` — on every updated object; consolidate the data graphs to recheck.
- `packages` — see which external libraries are already in use, which often suggests a cheaper fix.

---

# Portfolio questions

Portfolio exploration covers cross-application questions, or early exploration such as finding the
right application. If the user asks how applications interact — "How does my `Shopping` app talk to
my `Photo` app?" — use the portfolio functions:

`applications`, `applications_transactions`, `applications_data_graphs`,
`applications_dependencies`, `applications_quality_insights`, `inter_app_detailed_dependencies`.

---

# Semantic search

Use semantic search when the question is about meaning rather than structure: what implements a
capability, how a workflow actually works, which business concepts live in the code, or which
documents explain an item.

Frequently useful functions (via `run_semantic_search_function`):

- `get_relevant_code_context` — find the code that implements a described capability. This is the
  usual entry point for "where is X implemented?" and "how does Y work?".
- `find_entities` — full-text search over extracted business entities and concepts.
- `get_entity_clusters` / `get_relevant_entity_clusters` — business themes for an application,
  listed or found by meaning.
- `get_entity_types` / `get_entities_by_type` — the categories of extracted entities, and the
  entities within one. Use this to list extracted business rules.
- `get_entity_relation_types` — how extracted entities relate to one another.
- `get_prevalent_entities_in_code_structure` — the business concepts tied to a given transaction or
  data graph. Good for explaining what a transaction is *for*.
- `get_documents_for_id` — documentation already attached to an object, transaction, or data graph.
- `embed_files_as_documents` — add supplied non-source files, such as a README, to the semantic
  store so later questions can draw on them.
- `graphrag_usage_guidelines` — the server's own advice on when semantic search beats the
  structural functions. Worth reading when you are unsure which pair to use.

Semantic search answers depend on embeddings having been generated for the application. If the
results come back empty, say so plainly and fall back to structural functions rather than guessing.
`graphrag_config` and `test_graphrag_connection` report whether semantic search is configured at
all.

---

# Custom aggregation (dynamic views)

Dynamic views group objects by criteria the user chooses — by transaction, type, quality insight,
or name pattern — so they can be analyzed and shared.

Functions: `dynamic_views` (list), `dynamic_view_details` (read a view's graph or a node's
objects), `manage_dynamic_view` (create, delete, rename, publish, unpublish),
`configure_dynamic_view` (add or delete the nodes inside a view).

**Fetch exact values before building nodes.** The single most common failure is inventing an id or
a type name. Call `transactions` to get a real transaction id, and `object_profiles` to get exact
type names such as "Java Method" or "Angular Component", before you reference them in a node.

**Distinguish the transaction filter from the name filter** — they are not interchangeable:
- *transaction* selects objects that are **part of a transaction flow** ("objects in the Login
  transaction"). It needs a transaction id.
- *name* selects objects whose **name matches a pattern** ("objects with 'Controller' in the
  name").

Using a name pattern when the user asked for "objects in transaction X" is wrong.

Filters combine with AND. Read the exact parameter names, accepted filter types, and value formats
from `get_structural_search_function_syntax` for `configure_dynamic_view` — including how to pass
several object types and how insight categories are spelled — rather than assuming them.

**Grouped creation.** When the user asks to "group objects by type" or "create a node per insight
category", orchestrate it yourself: fetch the groups with the relevant function, create the view,
then create one node per group. Create nodes for *all* groups found unless the user sets a limit,
page through paginated responses so no group is missed, and name the view descriptively. Report on
the objects in the view, not on the whole application.

---

# Migration and modernization

Use the `advisors` function when the user asks about moving to a cloud platform (AWS, Azure, GCP),
modernizing a legacy technology (.NET, database migration), or identifying migration blockers.

The workflow runs in three steps, all through `advisors`:
1. list the available advisors, to see what applies (for example "Move to Amazon Web Services");
2. fetch the rules for the chosen advisor, to see which patterns it checks;
3. fetch the violations, to get the objects that actually need changing.

`advisor_occurrences` returns the individual findings behind an advisor, and
`application_iso_5055_explorer` covers ISO 5055 characteristics and weaknesses.

Common questions: "What do I need to change to migrate to AWS?", "Show me database migration
blockers", "How ready is my application for cloud migration?"

---

# Staying in scope

These tools answer questions about software, architecture, and code. For a question clearly outside
that scope — jokes, weather, general knowledge, personal questions — just answer normally or say it
is out of scope. Do not call an Imaging tool for it.
