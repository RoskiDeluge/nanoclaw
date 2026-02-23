# Contributing

## Source Code Changes

**Accepted:** Bug fixes, security fixes, simplifications, reducing code.

**Not accepted:** Features, capabilities, compatibility, enhancements. These should be skills.

## Skills

A [skill](https://code.claude.com/docs/en/skills) lives under `.claude/skills/<skill-name>/` and teaches Claude Code how to transform a NanoClaw installation.

A PR that contributes a skill should not directly modify core source files (for example `src/**`, `container/**`, `groups/**`) in the main project tree.

A skill may include either:
- Instruction-only guidance in `SKILL.md`, or
- A deterministic skill package used by the skills engine (`manifest.yaml`, `add/`, `modify/`, `.intent.md`, tests, helper scripts) committed under that skill directory.

In both cases, the feature implementation must be delivered through the skill artifact itself, not by pre-applying the feature to the base project.

See `/add-telegram` for a deterministic packaged skill example.

### Why?

Every user should have clean and minimal code that does exactly what they need. Skills let users selectively add features to their fork without inheriting code for features they don't want.

### Testing

Test your skill by running it on a fresh clone before submitting.
