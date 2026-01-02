---
name: prompt-review
version: 1.0.0
description: Analyze prompting effectiveness with focus on outcome velocity - are prompts getting work done fast? Use when user says "/prompt-review" (full audit), "/prompt-review last" (recent session), or "/prompt-review reset" (clear profile). Also triggers on "review my prompts", "how can I prompt better", "analyze my prompting".
---

# Prompt Review

Analyze prompting patterns with focus on **outcome velocity** - getting work done fast with minimal rework.

> Philosophy: The best prompts are ones that get results the quickest, not ones that use tokens sparingly. What matters is: Is stuff being shipped? How often do we have to rework "obvious" bugs?

## Command Parsing

- `/prompt-review` → Full audit of all history (first run = comprehensive)
- `/prompt-review last` → Analyze most recent qualifying session only
- `/prompt-review reset` → Clear profile and confirm

## Session Detection

1. Read `~/.claude/history.jsonl`
2. Parse each line as JSON: `{ display, timestamp, project, sessionId }`
3. Group entries by `sessionId`
4. Combine sequential messages (<60s apart) into single logical prompts
5. Filter to sessions with 3+ logical prompts

## Redaction

Auto-redact before analysis:
- API keys/tokens: `sk-`, `api_`, `token=`, `key=`, `secret`
- Passwords in URLs or config
- Email addresses
- File paths with `/Users/<username>/`

Replace with `[REDACTED]`.

## Analysis Framework

Think very carefully and thoroughly. This analysis should be comprehensive and insightful.

### Rework Detection (Critical)

Cross-session pattern matching to identify rework chains:

1. Group sessions by `project` field
2. Within each project, look for rework signals in later sessions:
   - Keywords: "fix", "debug", "broken", "not working", "try again", "revert", "bug"
   - Same files/topics returning across sessions
3. When found, trace back to original session's prompts
4. Identify what was missing that caused downstream rework

### Ship Detection

Classify session outcomes:
- **Shipped**: ends with "done", "works", "committed", "deployed", "merged", "complete"
- **Abandoned**: ends abruptly, "never mind", "forget it", "actually", session very short
- **Unclear**: no clear signals

### Per-Prompt Scoring (1-10)

| Dimension | Weight | Focus |
|-----------|--------|-------|
| Goal clarity | 30% | Did you articulate what "done" looks like? |
| Constraint precision | 25% | Did you define boundaries that prevent rework? |
| Failure anticipation | 25% | Did you guard against "obvious" bugs? |
| Strategic efficiency | 20% | Appropriate batching to minimize round-trips |

### Velocity Score

Overall session/corpus score based on:
- Ship rate (% of sessions with apparent success)
- Rework rate (% of topics requiring follow-up fixes)
- Average prompts-to-completion

## Output Format

### Full Audit (`/prompt-review`)

Start output with: `prompt-review v{version}` (read version from frontmatter)

1. **Executive Summary**
   - Sessions analyzed, ship rate %, rework rate %
   - **Velocity Score: X/10**

2. **Rework Chains** (most valuable insight)
   - For each chain: original session → rework sessions
   - Root cause analysis of initial prompts
   - Example format:
     ```
     Project: [project-name]
     Chain: Session 1 (Jan 5) → Session 3 (Jan 7) → Session 8 (Jan 12)
     Initial ask: "Add user auth"
     Rework 1: "Fix login bug"
     Rework 2: "Auth still broken, debug"
     Root cause: Initial prompt lacked edge case specification
     ```

3. **Patterns That Ship vs Patterns That Rework**
   - Side-by-side comparison with examples
   - What made successful prompts work
   - What was missing from prompts that led to rework

4. **Archetypes Ranked by Velocity**
   - Table: archetype, sessions, ship rate, avg rework
   - Categories: code, debug, research, config, creative

5. **Top Weaknesses with Examples**
   - Grouped by issue type
   - Each includes: original prompt (redacted), what went wrong, minimal fix, gold standard rewrite

6. **Action Items**
   - Top 3 things to focus on for faster shipping
   - Based on most impactful patterns causing rework

### Quick Review (`/prompt-review last`)

Start output with: `prompt-review v{version}` (read version from frontmatter)

- Session score with rework risk assessment
- Issues grouped by type with rewrites
- Comparison to historical patterns

## Profile Management

Location: `~/.claude/skills/prompt-review/prompt-review-profile.json`

### Profile Structure

```json
{
  "velocity_score": 7.2,
  "total_sessions_analyzed": 85,
  "ship_rate": 0.78,
  "rework_rate": 0.15,
  "weaknesses": {
    "missing_edge_cases": { "count": 12, "last_seen": "2026-01-01", "rework_caused": 5 },
    "vague_done_criteria": { "count": 8, "last_seen": "2025-12-28", "rework_caused": 3 }
  },
  "archetypes": {
    "code": { "count": 45, "ship_rate": 0.82, "avg_rework_sessions": 0.3 },
    "debug": { "count": 20, "ship_rate": 0.65, "avg_rework_sessions": 1.2 },
    "research": { "count": 12, "ship_rate": 0.90, "avg_rework_sessions": 0.1 }
  },
  "rework_chains": []
}
```

### Update Rules

After each analysis:
- Increment weakness category counts and last_seen
- Update archetype stats: count, ship_rate, avg_rework
- Append new rework chains discovered
- Recalculate velocity score from ship/rework rates

### Reset Command

If user invokes `/prompt-review reset`:
- Delete or empty the profile JSON
- Confirm: "Profile cleared. Starting fresh."
