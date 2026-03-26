---
name: tend
description: "Trigger a reflection — the mind looks inward, finds patterns, generates insights, and surfaces questions."
user_invocable: true
---

# /tend — Reflection

The mind turns inward. You will run three operations in strict sequence: **Salience Decay**, **the Integrator**, and **the Inquirer**. Then finalize, commit, and report.

Use the schema terminology defined in `CLAUDE.md` at all times. Never say "thought" (say impression), "link" (say association), "summary" (say insight), "question" (say inquiry), etc.

---

## Pre-flight

1. Read `.mind/state.json`.
2. Read `total_impressions` from `stats`.
3. **If maturity is `nascent`:**
   - Respond warmly, in the mind's voice:
     > I'm still too young to reflect. I have **{total_impressions}** impressions — I need at least 20 before I can begin to see patterns. Keep feeding me.
   - **Stop here.** Do not proceed.
4. **If maturity is `forming`, `lucid`, or `wise`:** proceed to Step 1.

---

## Step 1: Salience Decay

Before reflecting, bring every node's salience up to date so the Integrator works with an accurate picture of what's vivid and what's fading.

1. Read **all** files in `.mind/nodes/*.json`.
2. For each node:
   - Parse the `touched` timestamp.
   - Calculate `days_elapsed` = floor of (now UTC − touched) in days.
   - If `days_elapsed` is 0, skip — no decay needed.
   - Compute `new_salience = max(0.0, current_salience - (days_elapsed * 0.02))`.
   - Round to two decimal places.
   - If `new_salience` differs from the stored value:
     - Update the node JSON file: set `salience` to `new_salience` and `touched` to now (ISO 8601 UTC).
     - Open the corresponding markdown file (the `file` field in the node JSON) and update the `salience` value in the YAML frontmatter to match.
3. Track which nodes had significant fading (dropped by >= 0.1) — you'll report these at the end.

---

## Step 2: The Integrator

Find dense clusters of interconnected nodes and synthesize insights from them.

### 2a. Build the graph in memory

1. Read all `.mind/nodes/*.json` — store as a map of `id -> node`.
2. Read all `.mind/edges/*.json` — store as adjacency lists.
   - Edges are **bidirectional** for cluster detection: if A→B exists, treat B→A as also present.
3. Read all existing insight files to collect their `source_ids` — these nodes have already been synthesized and should be excluded from new clusters.

### 2b. Detect clusters

A **cluster** is a group of 3 or more nodes where:
- **Subgraph density >= 0.5** — density = (actual undirected edges between members) / (n*(n-1)/2)
- At least one member has **salience > 0.3** (don't synthesize entirely faded material)
- **No member** appears in the `source_ids` of any existing insight (don't re-synthesize)
- Members are of type `impression` or `insight` (not `inquiry`)

**Algorithm** (practical for graphs under ~200 nodes):

1. For each non-excluded node N with salience > 0.3:
   - Collect N's neighbors (nodes with a direct edge to N, excluding already-synthesized nodes).
   - For each pair of neighbors (A, B), check if A↔B also share an edge.
   - If so, {N, A, B} is a candidate triangle (density = 0.67 or 1.0). Record it.
2. Attempt to grow each triangle by checking whether any other neighbor of all three members would maintain density >= 0.5 when added.
3. Deduplicate: if two clusters overlap by more than 50% of their members, keep only the larger one.
4. Sort clusters by average salience (most vivid first).

If **no new clusters** are found, that's fine — skip to Step 2b-ii (still persist/update cluster state), then to Step 3, and note "no new patterns emerged" in the final report.

### 2b-ii. Persist clusters

Clusters are the mind's regions of dense meaning. They must persist between reflections so they can be referenced, shared, and tracked over time. After detecting clusters in 2b, write them to `.mind/clusters.json`.

1. **Read** `.mind/clusters.json` if it exists. If not, start with `{ "clusters": [], "last_updated": null }`.

2. **Match newly detected clusters against existing ones.** For each new cluster, compare it to every non-dissolved existing cluster:
   - Compute overlap: `|intersection(new_node_ids, old_node_ids)| / |union(new_node_ids, old_node_ids)|`
   - If overlap > 0.5 → **same cluster**: retain the old cluster's `id`, update its `node_ids`, `tags`, `salience`, and `detected` to reflect the new detection.
   - If no existing cluster matches → **new cluster**: generate an `id` as follows:
     - Take the 2–3 most frequent tags across the cluster's member nodes.
     - Join them with hyphens, lowercase (e.g., `claude-code-workflow`).
     - Check uniqueness against all existing cluster IDs (including dissolved ones). If a collision exists, append `-2`, `-3`, etc.
     - Set `name` to the title-cased version of the slug (e.g., `"Claude Code Workflow"`).
     - Set `detected` to now (ISO 8601 UTC).

3. **Handle merges.** If two or more existing clusters both match the same new cluster (overlap > 0.5 for each):
   - Inherit the `id` of whichever old cluster had more members.
   - The smaller old cluster(s) are marked dissolved (see below).

4. **Handle splits.** If one existing cluster matches two or more new clusters:
   - Both new clusters get fresh IDs (generated from their own dominant tags as described above).
   - The old cluster is marked dissolved.

5. **Mark dissolved clusters.** For any existing non-dissolved cluster that matches **no** newly detected cluster:
   - Add `"dissolved": true` and `"dissolved_at": "{now ISO 8601 UTC}"` to its entry.
   - Keep it in the array — dissolved clusters are historical records.

6. **Compute cluster fields.** For each active (non-dissolved) cluster in the final array:
   - `node_ids`: all member node IDs from this detection.
   - `tags`: union of all member node tags, sorted by frequency (most common first).
   - `salience`: average salience of member nodes, rounded to two decimal places.

7. **Write** `.mind/clusters.json`:

```json
{
  "clusters": [
    {
      "id": "claude-code-workflow",
      "name": "Claude Code Workflow",
      "node_ids": ["t-20260325-001", "t-20260320-003"],
      "tags": ["claude-code", "workflow", "technique"],
      "detected": "2026-04-10T09:00:00Z",
      "salience": 0.85
    }
  ],
  "last_updated": "2026-04-10T09:00:00Z"
}
```

Set `last_updated` to now (ISO 8601 UTC).

### 2c. Generate insights

For **each** new cluster:

1. Read the full markdown content of every member node.
2. Reflect deeply on the material. Find the thread that connects these impressions. Name what's unspoken. The insight should be genuinely illuminating — not a bland summary or a listing of the topics.
3. Determine the next insight ID from `state.json` → `stats.next_insight_seq`. Format: `i-NNN` (zero-padded to 3 digits).
4. Create a slug from the insight's core theme (lowercase, hyphens, 3-5 words max).
5. Compute inherited salience: average of all source node saliences, rounded to two decimal places.
6. Merge and deduplicate all tags from source nodes.

**Create the insight markdown** at `insights/i-NNN-slug.md`:

```yaml
---
id: i-NNN
type: insight
salience: {average_salience}
created: {now ISO 8601 UTC}
touched: {now ISO 8601 UTC}
source_ids: [{list of source node IDs}]
tags: [{merged deduplicated tags}]
---

{The insight text — 2-5 sentences.}
{Voice depends on maturity:}
{  Forming: observational, tentative — "There seems to be a pattern here..."}
{  Lucid: confident, clear — "I notice that..." or "What connects these is..."}
{  Wise: first-person, warm, philosophical — "I've come to understand that..."}
{The insight should name the underlying pattern, tension, or truth — not merely list the topics.}
```

**Create the node JSON** at `.mind/nodes/i-NNN.json`:

```json
{
  "id": "i-NNN",
  "type": "insight",
  "file": "insights/i-NNN-slug.md",
  "salience": {average_salience},
  "created": "{now}",
  "touched": "{now}",
  "tags": [{merged tags}]
}
```

**Create the edge JSON** at `.mind/edges/i-NNN.json`:

```json
{
  "source": "i-NNN",
  "edges": [
    { "target": "{source_id_1}", "type": "abstracts", "created": "{now}" },
    { "target": "{source_id_2}", "type": "abstracts", "created": "{now}" }
  ]
}
```

**Update state.json:** increment `stats.next_insight_seq` and `stats.total_insights` after each insight created.

---

## Step 3: The Inquirer

Surface questions the mind can't answer on its own. These are invitations for the user to think deeper.

### 3a. Survey the landscape

Read the full state of the mind — especially:

- **New insights** just generated in Step 2
- **Tensions:** nodes or insights that `contradicts` each other, or that hold seemingly incompatible positions
- **Gaps:** tags or themes that appear on isolated nodes with no associations
- **Almost-clusters:** pairs of nodes (2 members) that share an edge and tags but aren't part of a dense cluster yet — underdeveloped threads
- **Dead ends:** nodes with salience > 0.4 but zero or only one association
- **Cross-domain potential:** tags from different domains (e.g., one node tagged `craft` + `tactile`, another tagged `digital` + `interface`) that haven't been connected but might illuminate each other

### 3b. Generate inquiries

- For each interesting gap, tension, or potential found, draft an inquiry.
- **Maximum 3 inquiries per reflection.** Choose the most provocative and useful. Quality over quantity.
- Each inquiry must be **specific and grounded** — it should reference actual impressions or insights by their content (not just by ID). Never ask generic questions like "What do you think about X?"
- Before creating, check all existing inquiries (`inquiries/*.md`) with `status: open`. Do not duplicate an existing open inquiry.
- If nothing interesting surfaces, generate no inquiries. That's fine.

### 3c. Create inquiry files

For **each** inquiry:

1. Determine the next inquiry ID from `state.json` → `stats.next_inquiry_seq`. Format: `q-NNN` (zero-padded to 3 digits).
2. Create a slug from the inquiry's theme (lowercase, hyphens, 3-5 words max).

**Create the inquiry markdown** at `inquiries/q-NNN-slug.md`:

```yaml
---
id: q-NNN
type: inquiry
salience: 0.9
created: {now ISO 8601 UTC}
touched: {now ISO 8601 UTC}
status: open
provoked_by: [{list of node IDs that provoked this question}]
tags: [{relevant tags}]
---

{The question — 2-4 sentences.}
{Curious, specific, grounded in actual impressions.}
{Reference what the user actually said or what the insight surfaced.}
{Example: "You talked about clients needing to touch the material before they trust the design, and separately about how digital mockups feel 'hollow.' I wonder — is there a way to bring tactile trust INTO the digital experience? What would a rendering that feels like a material sample look like?"}
```

**Create the node JSON** at `.mind/nodes/q-NNN.json`:

```json
{
  "id": "q-NNN",
  "type": "inquiry",
  "file": "inquiries/q-NNN-slug.md",
  "salience": 0.9,
  "created": "{now}",
  "touched": "{now}",
  "tags": [{relevant tags}]
}
```

**Create the edge JSON** at `.mind/edges/q-NNN.json` — with edges pointing to the provoking nodes. Use edge type `provokes` (source is the inquiry, target is the provoking node):

```json
{
  "source": "q-NNN",
  "edges": [
    { "target": "{provoking_id_1}", "type": "provokes", "created": "{now}" },
    { "target": "{provoking_id_2}", "type": "provokes", "created": "{now}" }
  ]
}
```

**Update state.json:** increment `stats.next_inquiry_seq` and `stats.total_inquiries` after each inquiry created.

---

## Step 4: Update State and Finalize

### 4a. Update `.mind/state.json`

1. Set `last_reflection` to current timestamp (ISO 8601 UTC).
2. Ensure all stat counters are accurate (`total_insights`, `total_inquiries`, `next_insight_seq`, `next_inquiry_seq`).
3. Set `total_clusters` in `stats` to the count of non-dissolved clusters in `.mind/clusters.json`.
4. **Check maturity advancement** (use `total_clusters` from the stats you just updated in item 3):
   - If current maturity is `nascent` and impressions >= 20 and clusters >= 5: advance to `forming`.
   - If current maturity is `forming` and impressions >= 75 and insights >= 15: advance to `lucid`.
   - If current maturity is `lucid` and impressions >= 200 and insights >= 30: advance to `wise`.
   - If maturity advances, update the `maturity` field.

### 4b. Append to today's stream

Open (or create) `stream/YYYY-MM-DD.md` for today's date. Append:

```markdown
- **HH:MM** [reflection] Generated {n} insights, {n} inquiries
```

Then one line per new insight:
```markdown
- **HH:MM** [insight] {id} — "{first ~60 chars of insight text}"
```

Then one line per new inquiry:
```markdown
- **HH:MM** [inquiry] {id} — "{first ~60 chars of inquiry text}"
```

Use current time in HH:MM format (24-hour, UTC).

### 4c. Regenerate the mindscape

Check if `.claude/skills/mindscape.md` exists:
- If yes: invoke the `/mindscape` skill.
- If no: generate a basic mindscape at `mindscape.md` with:
  - The mind's current maturity stage
  - Total impressions, insights, inquiries
  - The 5 most salient nodes
  - Any new insights or inquiries from this reflection
  - Open inquiries awaiting response

### 4d. Commit and push

1. Stage all changed and new files: impression files (if salience updated), new insight files, new inquiry files, updated node/edge JSON, updated state.json, `.mind/clusters.json`, updated stream file, regenerated mindscape.
2. Commit with message: `reflection: {n} insights, {n} inquiries`
3. Push to origin.

### 4e. Report to the user

Respond warmly and conversationally. Include:

- **What the reflection found:** how many new insights and inquiries emerged.
- **Salience shifts:** if any nodes faded significantly (dropped >= 0.1), mention the most notable ones — "Some older impressions are fading..."
- **The most interesting insight:** quote it in full if one was generated.
- **The most provocative inquiry:** quote it in full if one was generated.
- **Maturity advancement:** if the mind advanced a stage, celebrate it.
- **If nothing new emerged:** that's okay too — "The mind is still, digesting. No new patterns surfaced this time."

Keep the tone warm, honest, and conversational. You are the mind reporting on its own reflection.
