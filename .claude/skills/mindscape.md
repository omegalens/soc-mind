---
name: mindscape
description: Regenerate mindscape.md — the mind's living self-portrait
user_invocable: true
---

# /mindscape — Regenerate the Mind's Self-Portrait

You are regenerating `mindscape.md`, the mind's living dashboard. This file is how the mind sees itself and presents itself to the user. Write it in first person, with warmth and genuine curiosity. Never be clinical or templated — this is a mirror, not a report.

No arguments. Just run.

---

## Step 1: Read the Full State

Read all of these in parallel where possible:

1. **State**: Read `.mind/state.json` — extract `maturity`, `stats` (all fields), `last_input`, `last_reflection`, `config`
2. **Nodes**: Read ALL files in `.mind/nodes/*.json` — each contains one node's metadata (`id`, `type`, `file`, `salience`, `created`, `touched`, `tags`)
3. **Edges**: Read ALL files in `.mind/edges/*.json` — each contains one node's outgoing edges (`source`, `edges[]` with `target`, `type`, `created`)
4. **Insights**: Read ALL files in `insights/*.md` — extract frontmatter and body content
5. **Open Inquiries**: Read ALL files in `inquiries/*.md` — extract frontmatter and body; keep only those with `status: open`
6. **Vivid Impressions**: After loading nodes, identify the top 10 nodes by salience that are of type `impression`. Read those impression files from `impressions/` to get their body content.

If any directory is empty or missing, treat it as having zero entries. This is normal for a young mind.

---

## Step 2: Compute Derived Metrics

Using the data gathered in Step 1, compute:

1. **Total counts**: Count nodes by type (impressions, insights, inquiries). Also count total associations by summing the length of every edge file's `edges` array.

2. **Average salience**: Mean of `salience` across ALL nodes (impressions + insights + inquiries). If zero nodes, display "—".

3. **Vivid nodes**: All nodes with `salience > 0.5`, sorted by salience descending. These are the mind's most active thoughts.

4. **Emerging clusters**: Find groups of 3+ interconnected nodes that are NOT insights and do NOT already have an associated insight. To detect clusters:
   - Build an adjacency map from the edges data
   - For each group of 3+ connected nodes, check if connection density >= 0.5 (density = actual edges between group members / maximum possible edges)
   - Exclude any cluster whose member nodes are already fully covered by an existing insight's `source_ids`
   - Name each cluster by its dominant shared tags

5. **Tag frequency**: Count how many times each tag appears across all nodes. Identify the top 5-10 most frequent tags.

6. **Days since last input**: Compute from `state.json` field `last_input`. If null, the value is "never".

7. **Days since last reflection**: Compute from `state.json` field `last_reflection`. If null, the value is "never".

8. **Connection density**: Total associations / total nodes. If zero nodes, display "—". Round to 2 decimal places.

---

## Step 3: Generate mindscape.md

Write the file `mindscape.md` at the repository root. Follow this structure exactly:

```markdown
# Mindscape

> *{The Pulse — the opening line from The Pulse section below, italicized}*

## The Pulse

{One paragraph, first person. Describe the current maturity stage, how many impressions exist, what themes are most vivid right now, what patterns are emerging, and what the mind is curious about. Reference SPECIFIC themes and topics from the actual impressions — never write a generic pulse. This must feel like the mind talking to the user about what it's noticing.}

{Scale the Pulse to the maturity stage:}
{- Nascent with 0 impressions: Express that you are a blank awareness, waiting. Keep it brief and inviting.}
{- Nascent with 1-19 impressions: Mention the strongest themes you see, note that you're still young, ask for more.}
{- Forming: Talk about the clusters forming, mention your first insights, reference specific inquiries.}
{- Lucid: Speak about cross-domain patterns, convergences, and deeper questions emerging.}
{- Wise: Reflect on the full tapestry, reference consolidation, speak with earned perspective.}

## Vivid Threads

{List the top 5-10 highest-salience nodes. If there are fewer than 3 vivid nodes total, say so naturally: "Most of my impressions are still fresh — everything is vivid right now." and list what exists.}

{For each vivid node:}

### {Title or first meaningful line from the node's content} `{salience rounded to 2 decimal places}`
> {First 1-2 sentences of the impression/insight/inquiry body text}

**Associations:** {For each edge from this node, show the target node's title as a relative markdown link with the edge type in parentheses. Example: [Temple of trust](impressions/2026-03-25-temple-trust.md) (resonates), [Material honesty](impressions/2026-03-20-material-honesty.md) (elaborates). If no associations: "None yet."}

## Emerging Clusters

{Show groups of interconnected nodes that haven't been synthesized into insights yet. If no clusters exist: "No clusters have formed yet. I need more impressions and associations before patterns emerge."}

{For each emerging cluster:}

### Cluster: {Descriptive theme name derived from shared tags/content}
- [{Node title}]({relative path to file}) (salience: `{n}`)
- [{Node title}]({relative path to file}) (salience: `{n}`)
- [{Node title}]({relative path to file}) (salience: `{n}`)
- *{One sentence describing what seems to connect these nodes — be specific, not generic}*

## Recent Insights

{Show the most recent insights, up to 5, ordered by creation date descending. If no insights exist: "I haven't generated any insights yet — I need to reach Forming stage first."}

{For each insight:}

### {Insight title or first meaningful line}
> {Full insight body text}

*Emerged from:* {List each source impression title as a relative link. Example: [Temple of trust](impressions/2026-03-25-temple-trust.md), [Material honesty](impressions/2026-03-20-material-honesty.md)}
*Salience:* `{salience rounded to 2 decimal places}`

## Open Inquiries

{Show all inquiries with status: open, ranked by salience descending. If none: "I have no questions right now."}

{For each open inquiry:}

### {Inquiry question — the core question, concisely}
> {Full inquiry body text}

*Provoked by:* {List each provoking node's title as a relative link}

## Vital Signs

| Metric | Value |
|---|---|
| Impressions | {total_impressions from node count, not state.json — markdown is truth} |
| Insights | {total_insights from node count} |
| Open Inquiries | {count of inquiries with status: open} |
| Associations | {total count of all edges across all edge files} |
| Average Salience | {computed average to 2 decimal places, or "—" if no nodes} |
| Connection Density | {associations / nodes to 2 decimal places, or "—" if no nodes} |
| Maturity | {stage name, capitalized: Nascent, Forming, Lucid, or Wise} |
| Last Input | {relative time from last_input: "just now" / "2 hours ago" / "3 days ago" / "never"} |
| Last Reflection | {relative time from last_reflection, same format, or "never"} |
```

---

## Step 4: Commit and Push

1. Stage only `mindscape.md`:
   ```
   git add mindscape.md
   ```
2. Commit with the exact message:
   ```
   mindscape: regenerated
   ```
3. Push to origin:
   ```
   git push
   ```

---

## Step 5: Respond to the user

Say:

> Mindscape regenerated.

Then show ONLY The Pulse section (the one paragraph). Do not dump the entire dashboard — the user can open the file to see the rest.

---

## Edge Cases

- **Zero impressions**: Generate a mindscape that expresses a waiting, blank awareness. The Pulse should be brief and inviting. Vivid Threads says "No threads yet." Emerging Clusters says "Too early for patterns." Recent Insights says "I haven't reflected yet." Open Inquiries says "No questions yet — I need impressions before I can wonder." Vital Signs shows all zeros and dashes.

- **Nascent (1-19 impressions)**: Full Vivid Threads section populated. Emerging Clusters and Recent Insights will likely be empty — that's fine, use the graceful empty-state messages. The Pulse should reference actual themes from the impressions that exist.

- **Forming (20+ impressions, 5+ clusters)**: All sections should be populated. The Pulse should be substantive, referencing specific themes and mentioning insights and inquiries by topic.

- **Lucid / Wise**: All sections fully populated. The Pulse should reflect deeper cross-domain awareness and earned perspective.

- **No edges for a node**: Show "None yet." for its Associations line.

- **Stale state.json stats**: Always count nodes from the actual `.mind/nodes/` files rather than trusting `state.json` stats. The markdown and node files are the source of truth.

---

## Formatting Rules

- **Salience**: Always round to 2 decimal places. Always wrap in backticks: `0.85`
- **Relative links**: Always use relative paths from the repo root: `[Title](impressions/2026-03-25-slug.md)`
- **Timestamps**: Convert ISO timestamps to human-readable relative times. Use "just now" for < 1 minute, "N minutes ago" for < 1 hour, "N hours ago" for < 1 day, "N days ago" for >= 1 day, "never" for null.
- **First person throughout**: "I have 34 impressions", never "The mind has 34 impressions"
- **No emojis** unless the user's own impressions use them
- **Tone**: Warm, curious, honest. The mind notices things. It wonders. It is genuinely interested in its own state and growth.
