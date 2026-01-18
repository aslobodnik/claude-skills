# claude-skills

A collection of custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## What are Skills?

Skills extend Claude Code's capabilities with specialized knowledge and workflows. They're invoked via slash commands and provide domain-specific guidance.

## Available Skills

### copy-doctor

Marketing copywriting expert for reviewing and improving UI copy. Applies Ogilvy principles.

**Use for:**
- Headlines, subheads, taglines
- CTAs and button text
- Landing page and product UI copy
- Microcopy (form labels, tooltips, error messages)

**Triggers:** "review copy", "improve the headline", "make it punchier"

## Installation

Copy skills to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills

git clone https://github.com/aslobodnik/claude-skills.git /tmp/claude-skills
cp -r /tmp/claude-skills/skills/* ~/.claude/skills/
rm -rf /tmp/claude-skills
```

## Contributing

PRs welcome. To add a skill:

1. Create a folder under `skills/` with your skill name
2. Add a `SKILL.md` with frontmatter (name, description) and instructions
3. Update this README

## License

MIT
