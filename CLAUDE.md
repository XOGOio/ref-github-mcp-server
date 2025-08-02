# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Build**: `go build ./cmd/github-mcp-server`
**Test**: `./script/test` (runs `go test -race ./...`)  
**Lint**: `./script/lint` (runs `gofmt` and `golangci-lint`) 
**Run local server**: `go run ./cmd/github-mcp-server stdio` (requires `GITHUB_PERSONAL_ACCESS_TOKEN` env var)

**Test with Docker**: 
```bash
docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN=your_token ghcr.io/github/github-mcp-server
```

**Update tool snapshots**: `UPDATE_TOOLSNAPS=true go test ./...`

## Architecture

This is a GitHub MCP (Model Context Protocol) server that provides AI tools access to GitHub's API. The architecture follows a modular toolset design:

**Entry Points**:
- `cmd/github-mcp-server/main.go` - CLI entry point using Cobra
- `cmd/mcpcurl/main.go` - Testing utility for MCP calls

**Core Server**:
- `internal/ghmcp/server.go` - MCP server configuration and setup
- `pkg/github/server.go` - GitHub-specific MCP server creation
- `pkg/github/tools.go` - Tool registration and toolset definitions

**Toolset Architecture**:
- `pkg/toolsets/` - Modular toolset system for organizing related tools
- Each toolset (repos, issues, pull_requests, actions, etc.) groups related functionality
- Tools can be selectively enabled/disabled via `--toolsets` flag or `GITHUB_TOOLSETS` env var
- Supports dynamic toolset discovery (`--dynamic-toolsets` flag)

**Tool Categories** (located in `pkg/github/`):
- `repositories.go` - Repository management (files, branches, commits)
- `issues.go` - Issue management and sub-issues  
- `pullrequests.go` - Pull request operations and reviews
- `actions.go` - GitHub Actions workflows and jobs
- `notifications.go` - GitHub notifications
- `search.go` - Code, repository, user, and issue search
- `discussions.go` - GitHub Discussions
- `gists.go` - GitHub Gist operations
- `context_tools.go` - User context (get_me)
- `code_scanning.go` - Code security alerts
- `dependabot.go` - Dependabot alerts
- `secret_scanning.go` - Secret scanning alerts

**Testing**:
- Unit tests use `testify` for assertions and `go-github-mock` for API mocking
- Tool schemas are snapshot-tested via `toolsnaps` utility in `internal/toolsnaps/`
- End-to-end tests in `e2e/` directory
- Each tool has corresponding `__toolsnaps__/*.snap` files for schema validation

**Configuration**:
- Supports GitHub Enterprise via `--gh-host` flag or `GITHUB_HOST` env var
- Read-only mode via `--read-only` flag or `GITHUB_READ_ONLY=1`
- Tool descriptions can be overridden via `github-mcp-server-config.json` or env vars with `GITHUB_MCP_` prefix

**Key Libraries**:
- `github.com/mark3labs/mcp-go` - MCP protocol implementation
- `github.com/google/go-github/v73` - GitHub REST API client
- `github.com/shurcooL/githubv4` - GitHub GraphQL API client
- `github.com/spf13/cobra` - CLI framework
- `github.com/spf13/viper` - Configuration management

The server can run both as a local binary and as a remote service hosted by GitHub, supporting various MCP host applications like VS Code, Claude Desktop, Cursor, and Windsurf.