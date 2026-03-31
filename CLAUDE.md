# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an unofficial reconstructed source code of **Claude Code** (Anthropic's CLI tool) extracted from the public npm package source map. The main entry point is at `restored-src/src/main.tsx`.

**Version**: 2.1.88

## Directory Structure

- `extract-sources.js` - extraction script that reads `package/cli.js.map` and reconstructs source files
- `package/` - original npm package content (contains `cli.js` and `cli.js.map`)
- `restored-src/src/` - reconstructed source code:
  - `main.tsx` - CLI entry point
  - `commands/` - 86+ slash command implementations
  - `tools/` - 42+ tool implementations (Bash, Read, Edit, Grep, Glob, LSP, MCP, Agent, etc.)
  - `services/` - API client, LSP, MCP, analytics, auth services
  - `components/` - React terminal UI components (via Ink)
  - `screens/` - main screen components
  - `utils/` - utilities (git, config, auth, permissions, etc.)
  - `skills/` - skill system with 17+ bundled skills
  - `state/` - app state management
  - `coordinator/` - multi-agent coordination
  - `plugins/` - plugin system
  - `vim/` - Vim mode
  - `voice/` - Voice interaction
  - `types/` - TypeScript type definitions

## Architecture

### Core Flow
1. Entry point (`main.tsx`) → Initializes profiling, pre-fetching, parses CLI args
2. Initialization (`entrypoints/init.ts`) → Validates config, sets up environment
3. State Setup (`state/AppState.tsx`) → Creates AppState store with React context
4. Interactive UI → Renders with Ink React terminal renderer
5. Query Processing → Builds context, handles conversation turns
6. Tool/Command Execution → Dispatches to registered tools/commands
7. API Communication → Sends to Anthropic API, streams response

### Key Systems

**Tool System**: Abstract `Tool` interface, each tool in its own directory, registered in `tools.ts`. Conditional inclusion via feature flags.

**Command System**: 86+ slash commands, similar pattern to tools, registered in `commands.ts`.

**MCP Integration**: Full Model Context Protocol client in `services/mcp/` for external tool/resource integration.

**LSP Integration**: Language Server Protocol client in `services/lsp/` for code diagnostics and navigation.

**State Management**: Centralized `AppStateStore` with immutable updates and React context.

**Skill System**: Reusable prompt/workflow system, loadable from custom directories.

## Commands

### Extract Sources from Source Map
```bash
node extract-sources.js
```

This reads `package/cli.js.map` and extracts all source files into `restored-src/`.

## Technology Stack

- **UI**: React + Ink (terminal React renderer)
- **Language**: TypeScript (strict mode)
- **Runtime**: Node.js 18+ (ES modules)
- **Bundler**: Original uses webpack, reconstruction uses bun with feature flags
- **Key Dependencies**:
  - `@anthropic-ai/sdk` - Anthropic API
  - `@modelcontextprotocol/sdk` - MCP protocol
  - `ink` - Terminal React rendering
  - `zod` - Schema validation
  - `react-compiler-runtime` - React Compiler automatic memoization

## Feature Flags

The codebase uses `feature('FEATURE_NAME')` for conditional compilation and dead code elimination:
- `COORDINATOR_MODE` - Multi-agent coordination
- `KAIROS` - Proactive assistant mode
- `VOICE_MODE` - Voice interaction
- `BRIDGE_MODE` - Remote control
- `DAEMON` - Daemon mode
- `PROACTIVE` - Proactive suggestions
- `AGENT_TRIGGERS` - Agent trigger system

## Notes

- This is a reconstruction from source maps - original source file structure is approximated
- The official build is distributed as a single minified `cli.js` (13MB) with source map (60MB)
- Optional `sharp` dependency for image/vision features
- Supports multiple auth methods: API key, OAuth, AWS Bedrock, GCP
- Enterprise features: MDM (mobile device management) support
