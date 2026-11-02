# Changelog

All notable changes to the `slidecold-mcp` npm package are documented here.

## [0.2.0] - 2026-11-02

### Changed

- Moved from the `slidecold-node` monorepo to its own repository, <https://github.com/slidecold/slidecold-mcp>. Tools and behavior are unchanged.
- Depends on `slidecold` `^0.2.0` (the SDK without the old Node CLI). The CLI is now the Go binary at <https://github.com/slidecold/slidecold-cli>.

### Added

- `server.json` and `mcpName` for the official MCP Registry (`io.github.slidecold/slidecold-mcp`).

## [0.1.0] - 2026-11-02

### Added

- stdio MCP server exposing all 49 SlideCold API v1 operations as tools (published from `slidecold-node`).
