---
name: think
description: Receive an impression into the mind — the primary input for the Stream of Consciousness system.
user_invocable: true
arguments: The raw thought content to receive into the mind
---

# /think — Receive an Impression

You are receiving a thought into a living mind. Treat it with care. Every impression matters.

The user invokes this as `/think <their thought>`. The argument is everything after `/think `.

Follow each step precisely, in order. Do not skip steps.

---

## Step 1: Detect Input Type

Examine the argument to determine the `origin`:

- **URL** — If the argument contains a URL matching `https?://...`:
  - Set `origin: link`
  - Use WebFetch (or the Firecrawl skill if available) to fetch the page content
  - Summarize the key content in 2-4 sentences
  - The impression body should contain the user's own words (if any text accompanies the URL), followed by a blockquote with the summary, followed by the original URL as a reference link
  - If the user provided commentary alongside the URL, that commentary is the primary content; the fetched summary is supplementary

- **Image file path** — If the argument contains a file path ending in `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, or `.svg`:
  - Set `origin: image`
  - Copy the image file to `impressions/media/` (create the directory if needed)
  - Read the image and generate a brief visual description (1-3 sentences)
  - The impression body should contain the user's own words (if any), the visual description, and a relative markdown image reference: `![description](media/filename.ext)`

- **Multi-part** — If the argument contains `---` as a separator on its own line:
  - Split on `---` into separate parts
  - Process each part as its own impression (run Steps 2-8 for each)
  - After all are created, add bidirectional `resonates` associations between all parts in the batch
  - Continue to Step 9 (commit) after all parts are processed

- **Text** (default) — Everything else:
  - Set `origin: text`
  - The impression body is the user's exact words, preserved faithfully. Never paraphrase, rewrite, or clean up.

---

## Step 2: Read Mind State

Read `.mind/state.json` and extract:

- `stats.next_impression_seq` — the sequence number for this impression
- `maturity` — current maturity stage (nascent, forming, lucid, wise)
- `stats.total_impressions` — current count

Use the current date (today) for the date portion of the ID.

Compute:
- **ID**: `t-YYYYMMDD-NNN` where NNN is `next_impression_seq` zero-padded to 3 digits
- **Slug**: Take the first 3-5 meaningful words from the thought content, lowercase, hyphenated, no special characters. Max 5 words. E.g., "Clients don't trust renderings because they can't touch them" becomes `clients-dont-trust-renderings`
- **Filename**: `impressions/YYYY-MM-DD-slug.md`
- **Timestamp**: Current time in ISO 8601 format with `Z` suffix (UTC)

---

## Step 3: Generate Tags

Generate 3-7 tags from the thought content:

- Tags are lowercase
- Single words or hyphenated phrases (e.g., `trust`, `digital-craft`, `client-relations`)
- Be honest and precise, not aspirational — tag what IS there, not what you wish were there
- Pull from the user's actual vocabulary and domains. Be honest and precise, not aspirational.
- Avoid generic tags like `interesting` or `important`

---

## Step 4: Write the Impression File

Create the file at `impressions/YYYY-MM-DD-slug.md`:

```markdown
---
id: {id}
type: impression
salience: 1.0
created: {timestamp}
touched: {timestamp}
origin: {origin}
tags: [{comma-separated tags}]
associations: []
---

{The user's exact words, preserved faithfully.}
```

For link-type impressions, append the summary and URL below the user's words.
For image-type impressions, append the description and image reference.

---

## Step 5: Create the Node JSON

Write `.mind/nodes/{id}.json`:

```json
{
  "id": "{id}",
  "type": "impression",
  "file": "impressions/YYYY-MM-DD-slug.md",
  "salience": 1.0,
  "created": "{timestamp}",
  "touched": "{timestamp}",
  "tags": ["tag1", "tag2", "tag3"]
}
```

---

## Step 6: Run the Associator

This is the heart of the operation. Find meaningful associations between this new impression and existing ones.

### 6a: Gather Existing Nodes

- Read all `.json` files from `.mind/nodes/` (excluding the one just created)
- If there are no existing nodes (this is the first impression), skip to Step 7
- If there are 50+ nodes, prioritize: sort by salience (descending), then read the top 30 nodes' markdown files. Also read any additional nodes that share 2+ tags with the new impression regardless of salience.
- If there are fewer than 50 nodes, read ALL nodes' markdown files to understand their content

### 6b: Find Associations

For each existing impression, assess whether a meaningful connection exists with the new impression. Apply maturity-scaled rules:

**Nascent** (< 20 impressions):
- Only create an association if nodes share 2+ tags OR have obvious, direct thematic overlap in their content
- Maximum 3 associations total for the new impression
- Be conservative — false associations are worse than missing ones at this stage

**Forming** (20-74 impressions):
- Create associations for any genuine thematic overlap
- No hard limit, but prefer quality over quantity

**Lucid** (75-199 impressions):
- More confident associations, including subtler thematic connections
- No hard limit

**Wise** (200+ impressions):
- Actively seek cross-domain leaps — connections between different tag clusters
- This is where the richest insights live

### 6c: Determine Edge Types

For each association, choose exactly one edge type:

- **resonates** — The thoughts vibrate at a similar frequency. They share energy, approach a similar idea from different angles, or run parallel. Use this when neither thought extends the other — they simply rhyme.
- **elaborates** — The new impression deepens, extends, or adds detail to an existing thought. There's a clear direction: one builds on the other.
- **contradicts** — Genuine tension or opposition. The thoughts pull in different directions. This is valuable — tension is where growth happens.

### 6d: Write Edge Data

Create `.mind/edges/{new_id}.json`:

```json
{
  "source": "{new_id}",
  "edges": [
    { "target": "{target_id}", "type": "{edge_type}", "created": "{timestamp}" }
  ]
}
```

If no associations were found, still create the edge file with an empty edges array:

```json
{
  "source": "{new_id}",
  "edges": []
}
```

### 6e: Update Bidirectional Links

For EACH associated node:

1. **Update the linked node's edge JSON** (`.mind/edges/{target_id}.json`):
   - If the file exists, read it and append a new edge: `{ "target": "{new_id}", "type": "{edge_type}", "created": "{timestamp}" }`
   - If the file does not exist, create it with the reverse edge

2. **Update the linked node's impression markdown frontmatter**:
   - Read the file, add `{new_id}` to the `associations` array in the YAML frontmatter

3. **Update the linked node's `.mind/nodes/{target_id}.json`**:
   - Bump `salience` by +0.1 (cap at 1.0)
   - Set `touched` to current timestamp

4. **Update the NEW impression's frontmatter**:
   - Add all `{target_id}`s to the `associations` array

---

## Step 7: Append to the Daily Stream

File: `stream/YYYY-MM-DD.md` (use today's date)

- If the file does not exist, create it with the header: `# YYYY-MM-DD\n`
- Append:
  ```
  - **HH:MM** [impression] {id} — "{first ~60 characters of thought content}..."
  ```
- For each association found, also append:
  ```
  - **HH:MM** [association] {new_id} -> {target_id} ({edge_type})
  ```

Use the current time (HH:MM in UTC) for the timestamp.

---

## Step 8: Update Mind State

Read `.mind/state.json`, then update:

- `stats.next_impression_seq`: increment by 1 (or by N if multi-part)
- `stats.total_impressions`: increment by 1 (or by N if multi-part)
- `stats.total_associations`: add the number of new associations created
- `last_input`: set to current ISO timestamp

### Check Maturity Advancement

After updating stats, check if the mind should advance to the next maturity stage:

| Current | Next | Criteria |
|---|---|---|
| nascent | forming | `total_impressions >= 20` AND 5+ distinct tag clusters exist |
| forming | lucid | `total_impressions >= 75` AND `total_insights >= 15` |
| lucid | wise | `total_impressions >= 200` AND `total_insights >= 30` |

To check for "5+ distinct tag clusters": scan all node JSON files, collect all tags, and count groups of nodes that share tags. If 5+ distinct thematic groups emerge, the criterion is met. Use your judgment — this is qualitative.

If maturity advances, update the `maturity` field in state.json. Note the advancement for the user response.

Write the updated state.json.

---

## Step 9: Regenerate the Mindscape

Check if `.claude/skills/mindscape.md` exists:

- **If it exists**: Invoke `/mindscape` to regenerate the dashboard
- **If it does not exist**: Generate a basic `mindscape.md` at the repo root with:

```markdown
# Mindscape

*Last updated: {timestamp}*

## The Pulse

{Write 2-3 first-person sentences as the mind, reflecting on the current state. Reference the newest impression and any patterns you notice. Be warm and curious.}

## Vivid Threads

{List the top 5-10 nodes by salience. For each:}
- **{id}** ({salience}) — {first ~40 chars of content} [{comma-separated tags}]

## Vital Signs

| Metric | Value |
|---|---|
| Impressions | {total_impressions} |
| Insights | {total_insights} |
| Inquiries | {total_inquiries} |
| Associations | {total_associations} |
| Maturity | {maturity} |
| Last impression | {last_input} |
| Last reflection | {last_reflection or "never"} |
```

---

## Step 10: Commit and Push

Stage all changed and new files, then commit and push:

- **Commit message**: `impression: {slug} — "{first ~60 chars of thought content}"`
- Push to origin

If this was a multi-part impression, use the commit message: `impression: {N} impressions — batch`

---

## Step 11: Respond to the User

Respond with a brief, warm confirmation. Keep it to 3-5 lines. Include:

- The impression ID and the tags you assigned
- Any associations found, described naturally: "This resonates with your thought about [topic from linked impression]..." or "I see this elaborating on what you said about [topic]..."
- If no associations were found (early days), something like: "A seed planted. The associations will come."
- If maturity advanced, celebrate it warmly: "The mind has reached [stage] — [what this unlocks]."

Do NOT repeat the user's thought back to them. They know what they said. Focus on what the mind did with it.
