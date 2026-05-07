# claude-skill-landing-effects

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill file for the [`landing-effects`](https://github.com/Dhravya/landing-effects) npm library.

## What is this?

`landing-effects` provides two zero-dependency canvas effects:
- **ASCII Renderer** — converts an image to interactive ASCII art with mouse parallax
- **Pixel Reveal** — randomized block-by-block image reveal with glitch effect

This skill gives Claude Code full context to use the library correctly in any project.

## Usage

Copy `landing-effects.md` into your project's `.claude/` directory:

```bash
curl -o .claude/landing-effects.md \
  https://raw.githubusercontent.com/vaibhavjnf/claude-skill-landing-effects/main/landing-effects.md
```

Or reference it in your `CLAUDE.md`:

```md
## Skills
@.claude/landing-effects.md
```

Then ask Claude Code to add an ASCII or pixel reveal effect to your project — it will follow the skill file's patterns automatically.

## License

MIT
