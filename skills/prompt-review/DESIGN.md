# Prompt Review Skill - Design Document

This document captures all design decisions, philosophy, and interview answers for future modifications.

## Core Philosophy

> **The best prompts are ones that get results the quickest, not ones that use tokens sparingly.**

What matters:
- Is stuff being shipped?
- How often do we have to rework "obvious" bugs?
- Are prompts getting work done fast over the medium term?
- Anyone can produce stuff quickly, but is it being shipped? And if so, how often do we rework?

**Key insight**: The first run on a big corpus of prompts is where we really want to focus - it needs to be "magical".

---

## Interview Q&A Summary

### Analysis Scope
- **Q**: Individual prompts vs conversation flow?
- **A**: Both with weighting (30% individual prompts, 70% conversation flow)

### Context Awareness
- **Q**: How to handle intentionally terse prompts after context established?
- **A**: Evaluate in context - judge if terseness was appropriate given prior messages

### Rating Criteria
- **Q**: What dimensions matter beyond clarity?
- **A**: ALL of these:
  - Specificity of constraints
  - Efficiency
  - Anticipating failure modes

### Attribution
- **Q**: Distinguish "bad prompt" vs "Claude misunderstood"?
- **A**: Always blame the prompt - focus on what YOU could have done better regardless of Claude's performance

### Long-term Tracking
- **Q**: Track patterns across sessions?
- **A**: Yes, persistent profile with weakness categories and prompt archetypes

### Privacy
- **Q**: Handle sensitive content?
- **A**: Redact and analyze - replace sensitive content with [REDACTED] before analysis

### Hook (Future)
- **Q**: Block until reviewed or async?
- **A**: Skip for v1 - focus on making a good manually-invoked skill first

### Trigger Threshold
- **Q**: Analyze every session?
- **A**: Minimum prompt count - only sessions with 3+ user messages

### Output Format
- **Q**: Suggest rewrites or just describe improvements?
- **A**: Rewritten examples - show both minimal fix AND gold standard side-by-side

### Experiment Mode
- **Q**: Different criteria for experimental prompts?
- **A**: Treat uniformly - apply same standards regardless

### Storage
- **Q**: Where to store profile?
- **A**: Same directory as skill (`~/.claude/skills/prompt-review/`)

### Scoring Scale
- **Q**: What rating scale?
- **A**: 1-10 numeric

### Profile Contents
- **Q**: What patterns to track?
- **A**: Weakness categories (with frequency count) + Prompt archetypes (ranked by average score)

### Score Weighting
- **Q**: Individual vs flow weighting?
- **A**: 30% prompts / 70% flow - conversation strategy dominates

### Duplicate Handling
- **Q**: Re-analyze prompts from previous runs?
- **A**: Re-analyze all - always analyze the full session fresh

### Profile Reset
- **Q**: Allow reset?
- **A**: Yes, full reset command

### Fallback Behavior
- **Q**: If last session too short?
- **A**: Find qualifying session - skip trivial, find most recent with 3+ prompts

### Session Scores
- **Q**: Individual scores, session score, or both?
- **A**: Both scores - individual prompt scores + weighted session average

### Rewrite Style
- **Q**: Minimal fixes or gold standard?
- **A**: Both for comparison - show minimal fix AND gold standard side-by-side

### Redaction Rules
- **Q**: Auto-detect or user-configured?
- **A**: Auto-detect common patterns (API keys, tokens, emails, file paths with usernames)

### Profile Updates
- **Q**: When to update profile?
- **A**: After each analysis - automatic, no confirmation needed

### Personalized Focus
- **Q**: Include top focus areas from profile?
- **A**: Yes, at the end - conclude with personalized recommendations

### Skill Name
- **Q**: What command?
- **A**: `/prompt-review` (user chose this over /prompt-coach, /improve, /review-prompts)

### Weakness Tracking
- **Q**: Frequency count or binary?
- **A**: Frequency count - track how often each weakness appears

### Archetype Scoring
- **Q**: Show best/worst archetypes?
- **A**: Yes, ranked - show average score per archetype

### Output Structure
- **Q**: How to organize per-prompt analysis?
- **A**: Grouped by issue type - organize prompts by primary weakness, not chronologically

### Analysis Persistence
- **Q**: Save analysis results?
- **A**: Profile only - no separate analysis logs, just profile updates

### Multi-message Handling
- **Q**: Sequential messages before Claude responds?
- **A**: Treat as single prompt - concatenate sequential user messages (<60s apart)

### Failure Mode Scope
- **Q**: Claude-specific or general LLM?
- **A**: Both with distinction - note which issues are Claude-specific vs universal

### Efficiency Criteria
- **Q**: What counts as inefficient?
- **A**: Redundant context, Over-specification, Under-batching (NOT verbose phrasing)

### Open-ended Prompts
- **Q**: How to evaluate intentionally open prompts?
- **A**: Score intent alignment - did you get what you wanted despite being open-ended?

### Data Limitation
- **Q**: history.jsonl only stores user prompts, not Claude responses - is inference OK?
- **A**: Yes, inference is fine - analyze flow based on user message patterns only

---

## Technical Details

### Data Source
- File: `~/.claude/history.jsonl`
- Format: JSONL (one JSON object per line)
- Each entry: `{ display, timestamp, project, sessionId }`
- **Limitation**: Only user prompts stored, not Claude's responses

### Session Detection
1. Parse history.jsonl
2. Group by sessionId
3. Combine sequential messages (<60s apart) into single logical prompts
4. Filter to sessions with 3+ logical prompts

### Rework Detection Algorithm
1. Group sessions by `project` field
2. Look for rework signals in later sessions:
   - Keywords: "fix", "debug", "broken", "not working", "try again", "revert", "bug"
   - Same files/topics returning
3. Trace back to original session's prompts
4. Identify root cause (what was missing)

### Ship Detection
- **Shipped**: ends with "done", "works", "committed", "deployed", "merged", "complete"
- **Abandoned**: ends abruptly, "never mind", "forget it", "actually", very short
- **Unclear**: no clear signals

### Velocity Score Formula
Based on:
- Ship rate (% of sessions with apparent success)
- Rework rate (% of topics requiring follow-up fixes)
- Average prompts-to-completion

### Profile Structure
```json
{
  "velocity_score": 7.2,
  "total_sessions_analyzed": 85,
  "ship_rate": 0.78,
  "rework_rate": 0.15,
  "weaknesses": {
    "missing_edge_cases": { "count": 12, "last_seen": "2026-01-01", "rework_caused": 5 }
  },
  "archetypes": {
    "code": { "count": 45, "ship_rate": 0.82, "avg_rework_sessions": 0.3 }
  },
  "rework_chains": []
}
```

---

## Commands

| Command | Behavior |
|---------|----------|
| `/prompt-review` | Full audit of ALL history (the "magical" comprehensive analysis) |
| `/prompt-review last` | Quick review of most recent qualifying session |
| `/prompt-review reset` | Clear profile entirely |

---

## Output Format (Full Audit)

1. **Executive Summary** - sessions analyzed, ship rate %, rework rate %, velocity score
2. **Rework Chains** - most valuable insight, traces initial prompts to downstream pain
3. **Patterns That Ship vs Patterns That Rework** - side-by-side comparison
4. **Archetypes Ranked by Velocity** - table with ship rate and avg rework per type
5. **Top Weaknesses with Examples** - grouped by issue, with minimal fix + gold standard
6. **Action Items** - top 3 things to focus on for faster shipping

---

## Future Considerations

### Hook (deferred to v2)
Originally discussed having this run at session start to review previous session. Deferred to focus on making the core skill excellent first.

### Improvement Trajectory
Considered tracking if user is getting better at specific dimensions over time. Decided to focus on weakness categories and archetype scoring instead.

---

## Files

- `~/.claude/skills/prompt-review/SKILL.md` - The skill definition
- `~/.claude/skills/prompt-review/DESIGN.md` - This document
- `~/.claude/skills/prompt-review/prompt-review-profile.json` - Created on first run
- `~/.claude/plans/proud-spinning-teacup.md` - Original planning spec
