---
name: share
description: Manage the membrane — control what parts of this mind are visible to communal minds.
user_invocable: true
arguments: Sharing command (e.g., "share cluster craft-philosophy", "unshare node t-20260325-042", "status")
---

# /share — The Membrane

The membrane is how this mind decides what to share with communal minds. Not a wall — a living boundary, like a cell membrane. It lets some things through and holds others private. This skill manages that boundary.

The user invokes this as `/share <command>`. The argument is everything after `/share `.

Use the schema terminology defined in `CLAUDE.md` at all times. Never say "thought" (say impression), "link" (say association), "summary" (say insight), "question" (say inquiry), etc.

Follow each step precisely, in order. Do not skip steps.

---

## Step 1: Read Current State

Read these files in parallel:

1. **`.mind/state.json`** — extract `maturity`, `stats`.
2. **`.mind/clusters.json`** — if it does not exist, note the absence. Cluster-level sharing commands will need clusters to be detected first.
3. **`.mind/sharing.json`** — if it does not exist, initialize it in memory as:
   ```json
   {
     "schema_version": 1,
     "rules": [],
     "communal_minds": []
   }
   ```
   You will write this file in Step 4.
4. **All `.mind/nodes/*.json`** — scan every node file. Store as a map of `id -> node` with `type`, `file`, `salience`, `tags`, and current `shared` value (if present).

---

## Step 2: Parse the Command

The argument after `/share ` determines the operation. Parse it against this table:

| Command | Operation |
|---|---|
| `share cluster {id}` | Add a cluster sharing rule: `{ "type": "cluster", "match": "{id}", "shared": true }` |
| `share cluster {id} types {type1} {type2} ...` | Add a cluster rule with type filter: `{ "type": "cluster", "match": "{id}", "shared": true, "filter": { "types": ["type1", "type2"] } }` |
| `unshare cluster {id}` | Remove/negate the cluster rule: remove any existing cluster rule for `{id}` from the rules array |
| `share tag {name}` | Add a tag sharing rule: `{ "type": "tag", "match": "{name}", "shared": true }` |
| `unshare tag {name}` | Remove the tag rule: remove any existing tag rule for `{name}` from the rules array |
| `share type {name}` | Add a type sharing rule: `{ "type": "type", "match": "{name}", "shared": true }` |
| `unshare type {name}` | Remove the type rule: remove any existing type rule for `{name}` from the rules array |
| `share node {id}` | Add a node-level override: `{ "type": "node", "match": "{id}", "shared": true }` |
| `unshare node {id}` | Add a node-level override: `{ "type": "node", "match": "{id}", "shared": false }` |
| `status` | Show current sharing summary — skip to Step 8 (Report) |
| `join {communal-id} {repo-url}` | Register with a communal mind (local only) |
| `leave {communal-id}` | Unregister from a communal mind (local only) |

If the command doesn't match any pattern, respond:

> I don't recognize that sharing command. Try `share cluster {id}`, `share tag {name}`, `share node {id}`, `unshare ...`, `status`, `join {id} {url}`, or `leave {id}`.

Then stop.

---

## Step 3: Validate

Before modifying anything, validate the command's target:

### Cluster commands (`share cluster` / `unshare cluster`)

1. Check that `.mind/clusters.json` exists. If it does not:
   > No clusters detected yet. Run `/tend` first to detect clusters, or use tag/type/node-level sharing.

   Then stop.

2. Read `.mind/clusters.json` and find the cluster with `id` matching `{id}`.
   - If no cluster with that ID exists, respond:
     > No cluster found with ID `{id}`. Available clusters: {list cluster IDs of non-dissolved clusters}.

     Then stop.
   - If the cluster exists but has `"dissolved": true`, warn:
     > Cluster `{id}` has been dissolved (it was detected in an earlier reflection but its members have drifted apart). Sharing it would have no effect. You can share the individual nodes instead.

     Then stop.

3. For type-filtered cluster rules (`share cluster {id} types ...`): verify each type is one of `impression`, `insight`, or `inquiry`. If any type is invalid, reject and list the valid types.

### Tag commands (`share tag` / `unshare tag`)

1. Scan all loaded nodes. Check that at least one node has the specified tag.
2. If no node has the tag, respond:
   > No nodes carry the tag `{name}`. Check existing tags with `/share status`.

   Then stop.

### Type commands (`share type` / `unshare type`)

1. Verify the type is one of: `impression`, `insight`, `inquiry`.
2. If not, respond:
   > `{name}` is not a valid type. Valid types are: impression, insight, inquiry.

   Then stop.

### Node commands (`share node` / `unshare node`)

1. Check that a node with ID `{id}` exists in the loaded nodes map.
2. If not, respond:
   > No node found with ID `{id}`.

   Then stop.

### Join command

1. Validate that `{communal-id}` and `{repo-url}` are both provided.
2. Check that the communal mind isn't already in the `communal_minds` array.
3. If already joined, respond:
   > Already registered with communal mind `{communal-id}`.

   Then stop.

### Leave command

1. Check that `{communal-id}` exists in the `communal_minds` array.
2. If not found, respond:
   > Not currently registered with `{communal-id}`.

   Then stop.

---

## Step 4: Update sharing.json

Based on the validated command, modify the in-memory sharing state:

### For `share` commands (cluster, tag, type, node)

1. Check if a rule with the same `type` and `match` already exists in the `rules` array.
   - If it does: replace it with the new rule (this handles updating a cluster rule to add a type filter, or changing a node override from shared to unshared).
   - If it does not: append the new rule.

### For `unshare` commands (cluster, tag, type)

1. Find any rule with the matching `type` and `match` in the `rules` array.
2. Remove it entirely. The absence of a rule means "not shared."

### For `unshare node {id}`

1. This is different from removing a rule. An explicit `unshare node` creates a node-level override with `"shared": false`. This is needed when a node falls within a shared cluster or tag but should be excluded.
2. Check if a rule with `type: "node"` and `match: "{id}"` exists:
   - If it does: update it to `"shared": false`.
   - If it does not: add `{ "type": "node", "match": "{id}", "shared": false }`.

### For `join`

1. Append to `communal_minds`:
   ```json
   {
     "id": "{communal-id}",
     "repo": "{repo-url}",
     "joined": "{now ISO 8601 UTC}"
   }
   ```
2. Note: this is local-only registration. The contributor must separately request membership from the communal mind operator. Remind the user of this.

### For `leave`

1. Remove the entry with matching `id` from `communal_minds`.

### Write the file

Write the updated sharing state to `.mind/sharing.json`.

---

## Step 5: Resolve Effective Sharing State

For every node in the mind, compute its **effective sharing state** by applying rules in specificity order. More specific rules override less specific ones:

1. **Node override** (most specific) — if a rule with `type: "node"` and `match: "{node_id}"` exists, its `shared` value wins. Done.
2. **Tag rule** — collect all tag rules where the node has a matching tag. If any matching tag rule has `shared: false`, the node is not shared (deny wins). If at least one has `shared: true` and none has `shared: false`, the node is shared.
3. **Type rule** — if a rule with `type: "type"` and `match` equal to the node's `type` exists, use its `shared` value.
4. **Cluster rule** — read `.mind/clusters.json` (if it exists). For each non-dissolved cluster, check if the node's ID is in that cluster's `node_ids`. If a cluster rule exists for that cluster:
   - If the cluster rule has a `filter.types` array, only share the node if the node's `type` is in that array.
   - Otherwise, share the node.
   - If multiple cluster rules match and they conflict, `shared: false` wins.
5. **Default** — if no rule matches at any level, the node is **not shared** (the `shared` field should be absent from the node).

Build a map of `node_id -> effective_shared` for all nodes. Track which nodes changed from their current state.

---

## Step 6: Update Frontmatter and Node JSON

For every node whose effective sharing state **changed** from its current state:

### When a node becomes shared (`shared: true`)

1. **Update `.mind/nodes/{id}.json`**: add or set `"shared": true`.
2. **Update the markdown file** (the `file` field in the node JSON): add `shared: true` to the YAML frontmatter. Place it after the `tags` field. If the field already exists, update its value.

### When a node becomes not shared (was `shared: true`, now absent or `false`)

There are two cases:

**a) Node has an explicit `unshare node` override (`shared: false`):**
1. Update `.mind/nodes/{id}.json`: set `"shared": false`.
2. Update the markdown frontmatter: set `shared: false`.

**b) Node is no longer shared because a rule was removed (no explicit override):**
1. Update `.mind/nodes/{id}.json`: **remove** the `shared` field entirely. Absent = not shared.
2. Update the markdown frontmatter: **remove** the `shared:` line entirely.

### Frontmatter editing

When updating YAML frontmatter in markdown files:

- Read the file.
- Parse the frontmatter (between the `---` delimiters).
- Add, update, or remove the `shared` field as needed.
- Place `shared` after `tags` and before `associations` (if present), or at the end of the frontmatter if those fields aren't there.
- Write the file back, preserving all other content unchanged.

---

## Step 7: Commit

Stage all changed files:
- `.mind/sharing.json`
- Any modified `.mind/nodes/*.json` files
- Any modified markdown files (impressions, insights, inquiries)

Commit with a descriptive message. Format:

```
membrane: {operation description}
```

Examples:
- `membrane: share cluster craft-philosophy`
- `membrane: unshare node t-20260325-042`
- `membrane: share tag technique (types: insight, inquiry)`
- `membrane: join communal mind claude-code-community`
- `membrane: leave communal mind claude-code-community`
- `membrane: share type insight`

Do **NOT** push. The membrane is a deliberate act — let the user decide when to push.

---

## Step 8: Report

### For `status` command

Show a comprehensive summary of the membrane state:

**1. Rules**

List all current rules, grouped by type:
```
Cluster rules:
  - craft-philosophy: shared (all types)
  - claude-code-workflow: shared (insights and inquiries only)

Tag rules:
  - technique: shared

Type rules:
  - insight: shared

Node overrides:
  - t-20260325-042: excluded (within shared cluster but explicitly unshared)
```

If no rules exist:
> No sharing rules configured yet. The membrane is fully closed.

**2. Effective sharing**

Compute and show:
```
Sharing N nodes across M clusters:
  - X impressions
  - Y insights
  - Z inquiries
```

If no clusters exist, omit the "across M clusters" part:
```
Sharing N nodes:
  - X impressions
  - Y insights
  - Z inquiries
```

**3. Communal minds**

List registered communal minds:
```
Registered communal minds:
  - claude-code-community (github.com/operator/claude-code-mind) — joined 2026-04-01
```

If none:
> Not registered with any communal minds yet. Use `/share join {id} {repo-url}` to register.

**4. Available tags** (for discoverability)

List all unique tags in the mind, sorted by frequency (most common first), with a count of how many nodes carry each tag. Limit to the top 20.

### For all other commands

After the operation completes, show:

```
Sharing N nodes across M clusters (X impressions, Y insights, Z inquiries)
```

Then briefly describe what changed:
- How many nodes gained `shared: true`
- How many nodes lost their `shared` status
- Any notable observations (e.g., "This shares all 7 nodes in the craft-philosophy cluster" or "This excludes t-20260325-042 from the shared cluster")

---

## Edge Cases

- **No clusters exist and user tries cluster sharing:** Guide them to `/tend` or to tag/type/node-level sharing instead.
- **Removing a rule that doesn't exist:** Respond gracefully — "No rule found for {type} `{match}`. Nothing changed."
- **Node in multiple clusters with different sharing rules:** The specificity precedence (Step 5) handles this. If a node-level override exists, it always wins.
- **Sharing everything:** The user might do `share type impression`, `share type insight`, `share type inquiry` — this effectively opens the membrane fully. That's their choice. Mention it: "The membrane is now fully open — all nodes are shared."
- **Empty mind:** If there are no nodes at all, respond: "The mind has no nodes yet. Use `/think` to add impressions first."
- **`join` is local-only:** Always remind the user that joining a communal mind here only registers the intent locally. The communal mind operator must also add this mind to their `commune.json` for the connection to be active.
