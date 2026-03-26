---
name: consult
description: "Query a communal mind — browse its mindscape, ask questions, explore cross-mind insights. Results stay local and firewalled."
user_invocable: true
arguments: The query to ask the communal mind, or "browse" to see the mindscape
---

# /consult — Reach Out to a Communal Mind

This skill lets the individual mind reach across to a communal mind and draw from its collective knowledge. You can browse the communal mindscape, ask questions, or explore specific themes.

Results are stored locally in `.communal/` which is gitignored. They never enter the mind's graph, never receive node IDs, and are never picked up by `/tend`. There is no feedback loop. The boundary between individual and communal is strict and deliberate — this is a read-only window, not a merge.

The user invokes this as `/consult <query>` or `/consult browse`. The argument is everything after `/consult `.

Use the schema terminology defined in `CLAUDE.md` at all times. Never say "thought" (say impression), "link" (say association), "summary" (say insight), "question" (say inquiry), etc.

Follow each step precisely, in order. Do not skip steps.

---

## Step 1: Identify Communal Mind

1. Read `.mind/sharing.json` and extract the `communal_minds` array.

   If `.mind/sharing.json` does not exist, respond:

   > No sharing configuration found. Use `/share status` to initialize sharing, or `/share join {id} {repo-url}` to connect to a communal mind.

   Then stop.

2. If `communal_minds` is empty or absent, respond:

   > Not connected to any communal minds. Use `/share join {id} {repo-url}` to connect.

   Then stop.

3. **If multiple communal minds are registered:** examine the argument for a communal mind ID prefix, formatted as `{mind-id}: {query}` — e.g., `/consult claude-code-community: what about X?`. If a prefix matching an entry in `communal_minds` is found, use that as the target mind and the text after `: ` as the query. If no prefix is found and there are multiple communal minds, respond:

   > Multiple communal minds registered. Specify one: `/consult {mind-id}: {query}`
   >
   > Available: {list the `id` values from `communal_minds`}

   Then stop.

4. **If only one communal mind is registered:** use it as the default target. The full argument is the query.

5. Note the target communal mind's `id` and `repo` URL for the steps ahead.

---

## Step 2: Clone or Pull

Ensure a local copy of the communal mind's repo is available.

1. Check whether `.communal/{mind-id}/repo/` exists as a directory.

   - **If it exists:** run `git -C .communal/{mind-id}/repo/ pull` to bring it up to date. If the pull fails (network error, auth issue), report the error and stop.

   - **If it does not exist:**
     - Run `mkdir -p .communal/{mind-id}/repo .communal/{mind-id}/queries`
     - Run `git clone --depth 1 {repo-url} .communal/{mind-id}/repo/`
     - If the clone fails (auth error, repo not found, network issue), report the error clearly. For private repos, note: the subscriber must have GitHub collaborator access — the communal mind operator adds subscribers as collaborators, and standard git credential helpers handle authentication. Then stop.

2. Once the repo is available, proceed to Step 3.

---

## Step 3: Read Communal State

Read the following from `.communal/{mind-id}/repo/`:

- **`mindscape.md`** — the communal mindscape (The Pulse, Vital Signs sections)
- **`.mind/state.json`** — communal maturity stage and stats
- **All `.mind/nodes/*.json`** — communal node index (for tag and keyword search)
- **All `insights/*.md`** — communal insight files (full content)
- **`inquiries/*.md` where `status: open`** — open communal inquiries only

Store these in memory for Steps 4 and 5.

---

## Step 4: Handle the Query

### If the argument is `browse` or empty:

Display the communal mind's current state:

1. Show the full **The Pulse** section and the **Vital Signs** table from `mindscape.md`.
2. List the **5 most recent communal insights** by `id` (most recent first). For each: show the ID and the first sentence of the insight text.
3. List **all open communal inquiries**. For each: show the ID and the first sentence of the inquiry text.
4. End with:

   > What would you like to explore?

### If the argument is a question or topic:

Synthesize an answer from communal knowledge.

1. **Search communal nodes** by matching the query's keywords and themes against:
   - Node tags (direct tag matches)
   - Node IDs and content keywords from the node JSON index
   - Insight and inquiry markdown content

2. **Read the full markdown** of all relevant nodes, insights, and open inquiries that match.

3. **Synthesize an answer.** Reflect on what the communal mind knows about this topic. Name patterns, resonances, and tensions that the communal knowledge reveals. Use the communal mind's voice: collective, observational, honest about uncertainty. Do not claim to speak for individual contributors.

4. **Cite sources** using communal node IDs throughout the synthesis:
   - Communal impressions: `cm-t-YYYYMMDD-NNN`
   - Communal insights: `ci-YYYYMMDD-NNN`
   - Communal inquiries: `cq-YYYYMMDD-NNN`
   - Example citation: "Based on `ci-20260410-007` and `cm-t-20260408-045`..."

5. If no relevant nodes are found, respond honestly:

   > The communal mind has no signal on this yet. It may not have been shared by any contributor, or the communal synthesis hasn't reached it.

---

## Step 5: Save Query Results

Write the synthesized result to `.communal/{mind-id}/queries/YYYY-MM-DD-{slug}.md`:

- **Date**: today's date
- **Slug**: 3-5 lowercase hyphenated words from the query

```yaml
---
query: "{the user's question, verbatim}"
communal_mind: {mind-id}
timestamp: {now ISO 8601 UTC}
sources: [{list of communal node IDs cited}]
---

{The synthesized answer, as written in Step 4.}
```

If the query was `browse`, save the browse output instead. Set `sources: []` for browse queries.

---

## Step 6: Enforce the Firewall

This is not a formality. The firewall is a design constraint that preserves the integrity of the individual mind.

**What `.communal/` is:**
- A read-only cache of communal knowledge, local to this machine
- Gitignored — it never enters the individual mind's version history
- A scratchpad for queries, not a graph

**What `.communal/` is NOT:**
- Part of the mind's graph — no node IDs are assigned
- Scanned by `/tend` — the Integrator and Inquirer read only `impressions/`, `insights/`, and `inquiries/`
- A source of automatic associations — nothing here touches `.mind/nodes/`, `.mind/edges/`, or any markdown file in the mind proper

**What this skill does NOT do:**
- Commit anything — `.communal/` is gitignored, so there is nothing to commit
- Write to `.mind/` in any form
- Create edges, update salience, or modify any existing node

After every query result (whether browse or a question), append this note:

> If this sparks a thought, capture it with `/think` in your own words. That's the only way communal knowledge enters your personal mind — through your own voice.

---

## Step 7: Append to the Daily Stream

Open (or create) `stream/YYYY-MM-DD.md` for today's date.

If the file does not exist, create it with the header `# YYYY-MM-DD`.

Append:

```markdown
- **HH:MM** [consult] {mind-id} — "{first ~60 chars of the query or "browse"}"
```

Use the current time in HH:MM format (24-hour, UTC).

This stream entry will be committed with the next `/think` or `/tend` operation — do not commit now.

---

## Error Handling

**Sharing.json missing or malformed:**
> The sharing configuration could not be read. Check that `.mind/sharing.json` exists.

**Repo clone fails (auth):**
> Could not access the communal mind's repo. For private repos, ensure you have collaborator access — ask the operator to add you on GitHub. Your local git credential helpers (keychain, gh auth, etc.) handle the rest.

**Repo clone fails (not found):**
> The repo URL registered for `{mind-id}` returned a 404. The URL may have changed. Use `/share leave {mind-id}` and `/share join {mind-id} {new-url}` to update it.

**Repo pull conflict:**
> The local communal cache has diverged from the remote. Delete `.communal/{mind-id}/repo/` and run `/consult` again to re-clone cleanly.

**No relevant content found:**
> The communal mind has no signal on this topic yet. It may emerge as more contributors share impressions in this area.

---

## Notes

- `.communal/` must be in the individual mind's `.gitignore`. If it is not present, add it before proceeding with Step 2. Check with `git check-ignore .communal/` and add the entry manually if missing.
- The `--depth 1` clone is intentional — only the latest state of the communal mind is needed. Full history is not required and would be slow.
- Multiple communal minds can be queried in sequence by invoking `/consult` multiple times with different prefixes.
- Auth for `/consult` (read access) is handled by standard git credential helpers. This is different from `/commune`, which uses `COMMUNE_GITHUB_TOKEN` because the operator needs programmatic write access across many repos.
