# claude-skills

A collection of custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## What are Skills?

Skills extend Claude Code's capabilities with specialized knowledge and workflows. They're invoked via slash commands (e.g., `/prompt-review`) and provide domain-specific guidance.

## Available Skills

### prompt-review (v1.0.0)

Analyze your prompting patterns with a focus on **outcome velocity** - getting work done fast with minimal rework.

**Philosophy**: The best prompts are ones that get results the quickest, not ones that use tokens sparingly. What matters is: Is stuff being shipped? How often do we have to rework "obvious" bugs?

#### Commands

| Command | Description |
|---------|-------------|
| `/prompt-review` | Full audit of all history |
| `/prompt-review last` | Analyze most recent qualifying session |
| `/prompt-review reset` | Clear profile and start fresh |

#### What It Analyzes

- **Rework chains**: Traces downstream bugs back to gaps in original prompts
- **Ship rate**: % of sessions that reach completion
- **Velocity score**: Overall effectiveness rating (1-10)
- **Weakness patterns**: Recurring issues like missing edge cases or vague criteria

#### Sample Output

```
Executive Summary
─────────────────
Sessions analyzed: 63
Ship rate: 43%
Rework rate: 57%
Velocity Score: 5.8/10

Top Rework Chain
────────────────
Project: lifelogger
Sessions: 14
Root cause: Initial architecture didn't account for timezone, cron timing
Chain: "digest sent 4x" → "reminders broken" → "transcription not working"

Top Weaknesses
──────────────
1. Reactive debugging (23 occurrences, caused 12 rework sessions)
2. Vague done criteria (22 occurrences, caused 6 rework sessions)
3. Missing edge cases (18 occurrences, caused 8 rework sessions)
```

## Installation

### Quick Install (Recommended)

Copy the skill to your Claude Code skills directory:

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Clone and copy
git clone https://github.com/aslobodnik/claude-skills.git /tmp/claude-skills
cp -r /tmp/claude-skills/skills/* ~/.claude/skills/
rm -rf /tmp/claude-skills
```

### Manual Install

1. Download the skill folder you want from `skills/`
2. Copy it to `~/.claude/skills/`

```bash
mkdir -p ~/.claude/skills/prompt-review
# Copy SKILL.md and DESIGN.md to ~/.claude/skills/prompt-review/
```

### Verify Installation

```bash
ls ~/.claude/skills/prompt-review/
# Should show: SKILL.md  DESIGN.md
```

## Usage

Once installed, start Claude Code and use the slash command:

```
/prompt-review
```

The skill reads your history from `~/.claude/history.jsonl` and analyzes your prompting patterns.

## How It Works

1. **Session detection**: Groups your history by session ID
2. **Rework detection**: Identifies "fix", "debug", "broken" patterns across sessions
3. **Ship detection**: Tracks sessions ending in "done", "works", "committed"
4. **Scoring**: Rates prompts on goal clarity, constraint precision, failure anticipation

### Scoring Dimensions

| Dimension | Weight | Focus |
|-----------|--------|-------|
| Goal clarity | 30% | Did you articulate what "done" looks like? |
| Constraint precision | 25% | Did you define boundaries that prevent rework? |
| Failure anticipation | 25% | Did you guard against "obvious" bugs? |
| Strategic efficiency | 20% | Appropriate batching to minimize round-trips |

## Privacy

The skill auto-redacts sensitive content before analysis:
- API keys/tokens (`sk-`, `api_`, `token=`)
- Passwords
- Email addresses
- File paths with usernames

## Profile Data

Your analysis profile is stored at:
```
~/.claude/skills/prompt-review/prompt-review-profile.json
```

Use `/prompt-review reset` to clear it.

## Contributing

PRs welcome! If you've built a useful skill, feel free to add it.

### Adding a New Skill

1. Create a folder under `skills/` with your skill name
2. Add a `SKILL.md` with frontmatter (name, description) and instructions
3. Optionally add a `DESIGN.md` documenting decisions
4. Update this README

## License

MIT
