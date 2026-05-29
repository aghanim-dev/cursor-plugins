# Changelog

All notable user-facing changes to the Aghanim Cursor marketplace and
its plugins are documented here. Format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); this marketplace
and its plugins use date-based version numbers of the form
`YYYY.MM.DD.N`, where `N` is a zero-indexed counter for multiple
releases on the same day.

## [Unreleased]

## [2026.05.29.0]

### Changed
- Newton MCP server URL updated to `https://mcp.aghanim.com/mcp` using the streamable HTTP transport.

## [2026.04.30.0]

### Added
- Documentation section in the README linking to the
  [Aghanim AI coding tools overview](https://docs.aghanim.com/ai-coding-tools/overview).

### Changed
- Intro line now calls this a "Cursor plugins repository" and links to
  `https://cursor.com/docs/plugins` (previously linked to the
  `#team-marketplaces` anchor).

## [2026.04.24.1]

### Changed
- Aligned `.cursor-plugin/marketplace.json` and `plugins/newton/.cursor-plugin/plugin.json` with the [cursor/plugin-template](https://github.com/cursor/plugin-template) schema: the marketplace plugin entry is now `{name, source, description}` only, and `homepage`/`repository` are dropped from the plugin manifest. No change to plugin behaviour or MCP configuration.

### Removed
- Dropped the marketplace README's Quick start section (team-marketplace install instructions) while the plugin is pending submission to the official Cursor plugin directory.

## [2026.04.24.0]

### Added
- `aghanim` marketplace catalog at `.cursor-plugin/marketplace.json`.
- `newton` plugin with Newton MCP server (`mcp__aghanim__*`) for Aghanim
  docs and API lookups, the `aghanim-webhooks-quick-start` skill, and
  the `/aghanim-webhooks-quick-start` slash command.

### Changed
- Reworded the `aghanim-webhooks-quick-start` skill to editor-neutral
  phrasing (e.g. "the assistant" instead of "Cursor", "your editor"
  instead of "Cursor") so the same text can be shared with the Claude
  Code variant. No behavioral change.
