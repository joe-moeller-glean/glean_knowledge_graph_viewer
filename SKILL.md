---
description:
  Apply when the user asks to generate, create, build, produce, or render
  the interactive HTML artifact, dashboard, interface for a 
  personalized enterprise knowledge graph / Glean context engine visualization. 
  Do NOT invoke for general questions or discussion about knowledge
  graphs, Glean Context, the Enterprise context engine, connected context, or similar
  concepts—answer those conversationally without running this skill or producing the HTML.
name: Enterprise Knowledge Graph Context Engine Visualizer
---
# Instructions

## When to use this skill (strict)

**Use this skill** only when the user’s intent is to **generate** (or create/build/produce/output) the **interactive `.html` artifact** described below—i.e. they want to fill the template in `assets/context_engine_graph_template.html` with real Glean-backed data.

**Do not use this skill** when the user is only asking **what** something is, how it works, product comparisons, architecture, definitions, or other **informational** topics—even if they mention “knowledge graph,” “Glean Context,” “enterprise context,” “context engine,” or closely related phrases. In those cases, respond normally and **do not** run data gathering for this template or emit the HTML artifact unless they also clearly ask you to generate it.

If ambiguous, **ask one short clarifying question** (“Do you want me to generate the interactive HTML graph, or explain the concept?”) instead of defaulting to generation.

## Goal

Produce one personalized `.html` file by filling `assets/context_engine_graph_template.html` with a graph payload derived from **the current user’s** Glean-accessible evidence. **Default time window: the last four weeks** for anything time-bounded (activity, recency, “recent” docs or meetings). If the user names another range, honor it (`after:` / `before:`, `updated:past_month`, meeting date bounds, etc.).

Do not fabricate named entities. Prefer empty arrays over guessed nodes.

## Payload shape

Match `assets/template_graph_data.json`: same top-level keys, nesting, and field conventions below. Omit no required key; use `[]` when there is no grounded content.

### Required top-level keys

`meta`, `user`, `llms`, `memories`, `people`, `teams`, `departments`, `projects`, `meetings`, `documents`, `orgNodes`, `enterpriseDocuments`, `partners`, `opportunities`, `accounts`

### Full document structure (JSON)

This is the object to populate (equivalent to `assets/template_graph_data.json`). Replace empty arrays with your nodes; keep every key present.

```json
{
  "meta": {
    "title": "Enterprise Knowledge Graph",
    "subtitle": ""
  },
  "user": {
    "id": "user-current",
    "label": "Current User",
    "subtitle": "Role",
    "kind": "user",
    "layer": "user",
    "group": "user",
    "desc": "Profile node for the current user.",
    "icon": "user"
  },
  "llms": [
    {
      "id": "llm-openai",
      "label": "OpenAI",
      "kind": "llm",
      "layer": "llm",
      "group": "llm",
      "desc": "LLM provider",
      "icon": "cpu"
    },
    {
      "id": "llm-anthropic",
      "label": "Anthropic",
      "kind": "llm",
      "layer": "llm",
      "group": "llm",
      "desc": "LLM provider",
      "icon": "cpu"
    },
    {
      "id": "llm-gemini",
      "label": "Gemini",
      "kind": "llm",
      "layer": "llm",
      "group": "llm",
      "desc": "LLM provider",
      "icon": "cpu"
    }
  ],
  "memories": [],
  "people": [],
  "teams": [],
  "departments": [],
  "projects": [],
  "meetings": [],
  "documents": [],
  "orgNodes": [],
  "enterpriseDocuments": [],
  "partners": [],
  "actions": [],
  "opportunities": [],
  "accounts": [],
  "edges": []
}
```

### Node objects (each element of the arrays above)

Use stable, unique string **`id`** values across the entire graph (URLs, issue keys, CRM ids, email hashes, etc.). Typical fields:

| Field | Required | Notes |
|--------|-----------|--------|
| `id` | yes | Unique across all nodes. |
| `label` | yes | Short primary title (template may truncate for display). |
| `subtitle` | recommended | Secondary line (role, date, source app, stage). |
| `desc` | recommended | Longer detail for the side panel. |
| `kind` | recommended | Semantic type; template may infer layout/styling (`user`, `llm`, `memory`, `person`, `team`, `project`, `meeting`, `doc`, `department`, `partner`, `datasource`, `tool`, `opportunity`, `account`, etc.). |
| `layer` | optional | `user`, `llm`, `personal`, `enterprise` — helps grouping when set. |
| `group` | optional | Finer bucket (e.g. `people`, `docs`); may be filled by normalization if omitted. |
| `icon` | recommended | For most nodes: built-in feather-style name (e.g. `user`, `calendar`, `file-text`, `cpu`). For 
| `iconUrl` | optional | Direct image URL for a datasource (or LLM) node; overrides map lookup when set. |
| `llmSlug` | **llms** (optional) | Same idea as **`connectorSlug`** for LLM rows; **`bySlug`** includes `openai`, `anthropic`, `gemini`. |
| `parent` | **only on `actions`** | String **`id`** of the `dataSources` node this tool belongs to. |
| `x`, `y` | optional | Numbers; layout usually assigns positions—omit unless you are overriding. |

The **`user`** object uses the same shape as other nodes (no `parent`).

**LLM nodes (`llms`):** the template resolves logos similarly via **`llmSlug`**, or **`id`** after `llm-`, or **`label`** against the same maps—use **`bySlug`** keys `openai`, `anthropic`, `gemini` when applicable.

## How to search for the data (Glean)

Use the smallest set of tools that still yields a dense, real graph. Iterate: broad search → narrow with filters → `read_document` only when snippets are insufficient (few high-value docs).

### Time and filters

- Default to **~four weeks**: combine `updated:past_month` or `after:YYYY-MM-DD` with keyword queries; for mail/calendar use the specialized tools’ date parameters when available.
- Scope to the user where it helps: owner/from/to/participant-style filters if your Glean query syntax supports them.
- Deduplicate by stable `id` (document URL, employee id, issue key, CRM id).

### `meta`

- Set `title` / `subtitle` from the run (e.g. user name + “Enterprise knowledge graph” and optional date range).

### `user`

- **`employee_search`**: current user’s profile — name, title, org path, location, manager/reporting hints. Build the single `user` node (`id` stable, e.g. `user-current`, plus real `label` / `subtitle` from title).

### `memories`

- **`read_memory`** (or equivalent): explicit preferences, standing context, saved facts. One node per memory with clear `label` / `desc`.

### `people`

- Collaborators and contacts: **`employee_search`** (peers, manager chain, frequent partners), **`meeting_lookup`** attendees, **`gmail_search`** correspondents, **search** for doc authors/commenters on docs the user touched. Stable `id` per person where Glean exposes one.

### `teams`

- **Search** for Slack channels, Google Chat spaces, Microsoft Teams, “team” or “group” artifacts the user belongs to or appears in; meeting series tied to a named team. Label with the team/channel name; `desc` can cite source (e.g. snippet or doc title).

### `departments`

- From **profile org path** and **search** for department pages, headcount/org charts, or internal “About our org” docs. Distinct from **`orgNodes`** if those are finer-grained org units; departments are broader buckets (e.g. “Engineering”, “Sales”).

### `projects`

- **Search** across Jira, Asana, Linear, Monday, etc.: epics, projects, initiatives the user owns, is assigned to, or is mentioned on. Use issue/project keys in `id` when available.

### `meetings`

- **`meeting_lookup`** (and calendar-oriented **search** if needed): recurring and one-off meetings in the window; titles, times, participants as `subtitle` / `desc`.

### `documents`

- **Search** with recency filters: docs the user edited, owns, or that show up in their recent work; include both **personal**-relevant files and key **shared** docs. Prefer canonical URLs for `id`.

### `orgNodes`

- Org-chart-like nodes: reporting lines, cost centers, business units, or “org tree” entities from **employee_search** + **search** (Workday, BambooHR, internal wiki org pages). More granular than **`departments`** when the index exposes that structure.

### `enterpriseDocuments`

- **Search** for company-wide policies, handbooks, OKRs, leadership updates, and other **not user-only** canonical docs that represent shared enterprise context.

### `partners`

- **Search** Salesforce and partner-facing content: customers, vendors, resellers, named partner programs. Ground names in snippets or structured fields; use CRM ids in `id` when present.

### `opportunities`

- **Search** Salesforce for open/pipeline opportunities and the user or their team touches in the window. Use CRM-stable ids; `subtitle` for stage, amount, or owner when present.

### `accounts`

- **Search** Salesforce for named accounts the user or their team touches in the window.

## Save the graph data as `knowledge_graph.json`

Before touching the HTML, write the populated graph object to **`knowledge_graph.json`** in the project root. The file must be valid JSON (strict — quoted keys, no trailing commas, no comments). It must contain all required top-level keys from **Full document structure (JSON)** above, with every array either populated or `[]`.

```json
// knowledge_graph.json — example skeleton (replace arrays with real nodes)
{
  "meta": { "title": "...", "subtitle": "..." },
  "user": { "id": "user-current", ... },
  "llms": [...],
  "memories": [...],
  "people": [...],
  "teams": [...],
  "departments": [...],
  "projects": [...],
  "meetings": [...],
  "documents": [...],
  "orgNodes": [...],
  "enterpriseDocuments": [...],
  "partners": [...],
  "actions": [...],
  "opportunities": [...],
  "accounts": [...],
  "edges": [...]
}
```

This file serves as the ground-truth record of what was found in Glean. It is separate from the HTML so it can be inspected, version-controlled, or re-ingested without re-running Glean searches.

## Install the graph JSON in the HTML template

**Read `assets/context_engine_graph_template.html` first** so you know the exact variable name and location before editing.

Locate the variable assignment in the main `<script>` block. It looks like one of:

```js
window.KNOWLEDGE_GRAPH_DATA = { ... };
```

Replace the **entire** existing object literal — from the opening `{` to the closing `}` — with the object you built (the same data you wrote to `knowledge_graph.json`). In the HTML file you may use a JavaScript object literal (unquoted keys where valid); it does not have to be strict JSON.

**Do not change anything else in the file.** Preserve all other script content, styles, markup, and comments exactly as they are.

After the replacement, verify:
1. The variable name is unchanged (e.g. `window.KNOWLEDGE_GRAPH_DATA =`).
2. The object closes with `};` (semicolon-terminated assignment).
3. No stray characters were introduced before or after the object.

Save the edited file as the `.html` artifact for the user, using the output filename they requested (default: `knowledge_graph.html`).
