# Changelog

All notable changes to this guide are recorded here, in
[Keep a Changelog](https://keepachangelog.com) format, newest first.

## [Unreleased]

### Fixed

- `MODELS-EFFORT-CONTEXT.md`: corrected our starting context and working room (about 100K, not
  130–160K), and that `auto` compaction is per model only.

- `templates/global-CLAUDE.md`: the optional-modules example pointed at `~/.claude/rules/`, which
  Claude Code loads at launch, so the example defeated its own purpose. It now uses
  `~/.claude/guides/` and says why.

### Added
- `MODELS-EFFORT-CONTEXT.md`: measured window sizes and a sizing rule, what auto-sizing does
  and doesn't exist, how to audit plugins and MCP servers, and lean custom agents.
- `MODELS-EFFORT-CONTEXT.md`: choosing Haiku, Sonnet, Opus or Fable by how much a step
  decides; what effort levels do and how to set them; context window sizing (we cap
  auto-compaction at 300K); what compaction keeps and loses, and when to compact;
  orchestrator and advisor patterns, with Anthropic's measured caveats; a rule that gives
  Claude authority to right-size agents.
- `GAME-THEORY.md`: how cheap agent work flips old best practices, moves the bottleneck
  to review and verification, and changes incentives for agents and attackers.
- `CROSS-DOMAIN.md`: borrowing solutions from other industries through their terms of art
  (TRIZ, analogical transfer), with a table of the same problem under many names.
- `STATE-MACHINES.md`: everyday state machines (turnstiles, elevators, flights, parcels)
  and the unhandled cells behind familiar frustrations.
- Lessons 27–29. The global template now covers four model tiers, compaction, cross-field
  search and re-pricing habits.

### Changed
- Lesson 12 now covers four model tiers (Haiku, Sonnet, Opus, Fable) and how sub-agents
  inherit the session's model and effort.
- `STATE-MACHINES.md`: ask the agent for a finite state machine (states, events, a full
  state × event table and a Mermaid diagram) to find edge cases before they happen and to
  check that agent and developer understand the problem the same way. Adds lesson 26 and
  a matching rule in the global template.
- `PRIOR-ART-AND-SKILLS.md`: research existing libraries and platform features before
  building, use and adopt skills deliberately, and use Rarefaction before grepping.
  The global and project templates and the setup instructions now carry these rules.
- First public version: starter instructions for an agent (`START-HERE-FOR-CLAUDE.md`),
  25 hard-won lessons, global and per-project `CLAUDE.md`/`AGENTS.md` templates, hardened
  GitHub Actions templates and a PR template, and notes on code-intelligence MCP servers
  (Rarefaction, Serena, graphify), GitHub flow with free CI, DigitalOcean and OpenRouter.
