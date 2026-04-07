---
description: Generates a personalized interactive HTML artifact that visualizes a user's enterprise knowledge graph in a 2D layered layout and compares Glean's connected context engine against a flatter competitor context model.
name: Enterprise Knowledge Graph Context Engine Visualizer
---
# Instructions

Read the `html_artifact_guide` skill first.
## Goal

Create one interactive `.html` artifact with a toggle between:

1. **Glean's Context**
2. **Competitor's Context**

Always use the bundled template in `assets/context_engine_graph_template.html` as the source of truth. Do not redesign from scratch unless the user explicitly asks.
## Brand and visual source of truth

Use these references as the design source of truth:

- `https://drive.google.com/file/d/1a-6QRo6n_dOpYOZiTXKQgaLsKt4wf9oJ`
- `https://www.glean.com/brand-resources`

Use `glean_document_reader` for those URLs when you need to confirm branding details.
### Required brand constraints

- Use only the **Glean primary palette**
    - Primary Blue: `#343CED`
    - Bright Green: `#D8FD49`
    - Oatmeal: `#F6F3EB`
    - Black: `#000000`
    - White: `#FFFFFF`
- Prefer **DM Sans** as the fallback typeface when PolySans is not available.
- Do not introduce off-brand colors.
- Keep the experience fast and lightweight: all graph rendering must be **2D**, never 3D.
## Data-specific constraint

The bundled HTML is a reusable template. Each run of the skill must populate it with the **current user’s** data.
## Core interaction requirements

The final HTML must:

- be fully **2D**
- support **pan and zoom**
- render quickly on click
- update the right-hand detail panel when a node is clicked
- wrap node text when needed
- truncate node text longer than **50 characters**
- use larger nodes and larger labels for readability
- keep nodes neatly arranged within their layer
- avoid node overlap
- fill horizontal space with more columns where possible
- keep nodes out from under section subheaders
- keep tool nodes arranged around datasource nodes like a **sun with rays**
## Gather user context

Build the graph from real user-specific evidence whenever possible.

Use the smallest tool set that gives enough real structure:

- `employee_search` for the current user profile, title, org, collaborators
- `memory` for explicit user memories and preferences
- `user_activity_retrieve` for recent activity across apps
- `glean_search` for documents, meetings, initiatives, teams, partners, trackers, and datasource evidence
- `glean_document_reader` only for a small number of key documents when titles/snippets are insufficient
- `tool_search` to discover datasource-related MCP tools or action-pack tools where needed
## Normalize to the bundled data model

Create a payload matching `references/data-model-example.json`.
### Required top-level keys

- `meta`
- `user`
- `llms`
- `memories`
- `people`
- `projects`
- `meetings`
- `documents`
- `orgNodes`
- `enterpriseDocuments`
- `partners`
- `dataSources`
- `actions`
- `edges`
## Minimum graph richness

For the **Glean's Context** view, aim for:

- at least **100 nodes**
- at least **2.5x as many edges as nodes**

Use real evidence first. Increase density by:

- including many collaborators, teams, projects, meetings, docs, partners, and datasources
- attaching tool/action nodes to datasources
- adding many-to-many edges where a real relationship is supported
- connecting documents to sources, projects, meetings, people, org nodes, and partners when supported

Do **not** invent fake named people, documents, or partners.
## View requirements
### Glean's Context

This should remain a graph, but it must be organized into clean vertical layers:

1. **LLM layer** at the top
2. **User layer** with the profile node beneath it
3. **Personal Knowledge Graph** in the middle
4. **Enterprise Knowledge Graph** at the bottom

Additional required behavior:

- Add a ring around the user node labeled **User Memories**
- Include rectangles for **OpenAI**, **Anthropic**, and **Gemini**
- Draw lines from the user node to those LLM nodes
- Most moving dots should bounce through the personal and enterprise graph
- Some dots should move between the personal and enterprise layers
- Fewer dots should travel between the user and the LLMs
- Moving points should **generate from the profile node**
- When a moving point returns from visiting an LLM, a new moving point should generate going toward the graph
### Competitor's Context

Render a flatter comparison:

- single profile node in the center
- datasource nodes around it
- datasource nodes should still use app icons
- a single rectangle labeled **LLM provider**
- a single moving dot going in and out from the profile
- no rich personal or enterprise web
- no deep multi-hop context graph
## Datasource and tool rules

- All datasource nodes should use their app icon as the node icon
- Find related tools for each datasource
- These may come from MCP servers, action packs, or other datasource-specific tool surfaces
- Add those tool nodes around the datasource node
- Connect them with sun-ray style edges
- Never invent unsupported tools
## Asset reuse rules

Always reuse `assets/context_engine_graph_template.html`.

To create a personalized output:

1. Copy the template
2. Replace the `TEMPLATE_GRAPH_DATA` object with the normalized payload for the current user
3. Keep the interaction model, right panel, layered layout, and toggle behavior
4. Save the instantiated result as the final `.html` artifact
## Output behavior

Return exactly one `.html` artifact for the current user.

After the artifact, add one short sentence telling the user to use the Preview eye toggle to render it.
