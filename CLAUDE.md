# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

x402 is a multi-language payment protocol monorepo implementing HTTP-native blockchain payments using the 402 status code. The repository contains implementations in TypeScript, Python, Go, and Java, along with framework-specific middleware packages and comprehensive E2E tests.

## Repository Structure

### Language Implementations

- **typescript/**: Primary implementation with pnpm workspace monorepo
  - `packages/x402/`: Core protocol library with modular exports
  - `packages/x402-{express,hono,next,axios,fetch}/`: Framework-specific integrations
  - `packages/coinbase-x402/`: Coinbase facilitator implementation
  - `site/`: Documentation and ecosystem site

- **python/**: Python implementation using hatch + pytest
  - `python/x402/`: Core library with FastAPI/Flask support

- **go/**: Go implementation using standard modules
  - `go/pkg/`: Core types and facilitator client
  - `go/pkg/gin/`: Gin framework middleware

- **java/**: Java implementation using Maven
  - Servlet-based middleware

- **e2e/**: Cross-language end-to-end test suite
  - Tests all client-server-facilitator combinations across languages

- **examples/**: Working examples for each language and framework

- **specs/**: Protocol specifications and scheme definitions

## Core Architecture

### Three-Party Model

The x402 protocol operates with three distinct roles:

1. **Client**: Entity paying for a resource (e.g., axios, fetch, httpx)
2. **Resource Server**: HTTP server providing paid content (e.g., Express, FastAPI, Gin)
3. **Facilitator**: Third-party service handling payment verification and settlement

### Payment Flow

1. Client requests resource → Server responds with 402 + payment requirements
2. Client creates payment payload and includes it in `X-PAYMENT` header
3. Server verifies payment (via facilitator or locally)
4. Server settles payment (via facilitator or locally)
5. Server returns resource with `X-PAYMENT-RESPONSE` header

### Core Package Structure (TypeScript)

The `typescript/packages/x402` package uses modular exports for different concerns:

- `x402/client`: Client-side payment creation and handling
- `x402/verify`: Payment verification utilities
- `x402/facilitator`: Facilitator server implementation
- `x402/schemes`: Payment scheme implementations (exact, etc.)
- `x402/shared`: Common utilities and types
- `x402/shared/evm`: EVM-specific shared code
- `x402/paywall`: Pre-built browser paywall widget
- `x402/types`: TypeScript type definitions

This modular structure allows importing only what's needed: `import { ... } from 'x402/client'`

### Schemes

Payment schemes define how funds move on-chain. The protocol is extensible to support different payment patterns:

- `exact`: Transfer exact amount (implemented for EVM, SVM, SUI)
- Future schemes could support streaming, usage-based, etc.

Each scheme may have different implementations per blockchain network.

## Development Commands

### TypeScript (from `typescript/` directory)

```bash
# Install dependencies
pnpm install

# Build all packages (uses Turbo for caching)
pnpm build

# Run tests across all packages
pnpm test

# Lint and format
pnpm lint          # Fix linting issues
pnpm format        # Format code
pnpm lint:check    # Check without fixing
pnpm format:check  # Check formatting

# CRITICAL: Rebuild paywall after core changes
cd packages/x402
pnpm build:paywall  # Must run and commit generated files

# Individual package operations
cd packages/x402
pnpm test          # Run tests for this package
pnpm test:watch    # Watch mode
```

### Python (from `python/x402/` directory)

```bash
# Install dependencies
pip install -e ".[dev]"

# Run tests
pytest

# Lint/format
ruff check .       # Check for issues
ruff check --fix . # Auto-fix issues
ruff format .      # Format code

# Build package
python -m build
```

### Go (from `go/` directory)

```bash
# Install dependencies
go mod download

# Run tests
go test ./...

# Build
go build ./...

# Run linter (if installed)
golangci-lint run
```

### Java (from `java/` directory)

```bash
# Build and test
mvn clean install

# Run tests only
mvn test

# Run linting
mvn verify -P lint

# Check code coverage
mvn test -P coverage
```

### E2E Tests (from `e2e/` directory)

The E2E test suite validates client-server-facilitator communication across all language implementations.

```bash
# Development mode (recommended)
pnpm test -d              # Run on testnet
pnpm test -d -v           # Verbose logging

# Filter by language
pnpm test -d -ts          # TypeScript only
pnpm test -d -py          # Python only
pnpm test -d -go          # Go only
pnpm test -ts -py         # TypeScript and Python

# Filter by implementation
pnpm test -d --client=axios      # Specific client
pnpm test -d --server=express    # Specific server
pnpm test -d -ts --server=next   # Next.js development

# Production testing
pnpm test --prod=true     # Use CDP facilitator
pnpm test --network=base  # Test on base mainnet
```

## Critical Development Workflows

### When Modifying Core x402 Package

**IMPORTANT**: If you update `typescript/packages/x402/src/`, you MUST rebuild the paywall:

```bash
cd typescript/packages/x402
pnpm build:paywall
git add src/paywall/generated/
```

The paywall is a pre-built browser widget that gets bundled into the package. The generated files must be committed as part of your changes.

### Adding New Payment Schemes

1. Create spec in `specs/schemes/{scheme-name}/`
2. Open PR with spec for review by CDP legal/security
3. After spec approval, implement in core package(s)
4. Add tests for new scheme
5. Update facilitator to support scheme

### Cross-Language Development

When adding features that span multiple languages:

1. Update specs first to define the interface
2. Implement in TypeScript (primary reference)
3. Port to other languages following language-specific best practices
4. Add E2E tests covering all combinations
5. Ensure backward compatibility with existing schemes

### Contributing Requirements

- All commits must be signed (git commit -S)
- PRs require approval from CDP Engineering team
- New schemes undergo legal and security audit
- Language libraries should follow language-specific best practices

## Package Manager

This repository uses **pnpm** (v10.7.0+) for TypeScript packages. The root `package.json` enforces this via `packageManager` field. Node.js v18+ is required.

## Protocol Versioning

The protocol uses `x402Version` field in all payloads. Current version is defined in specs. Implementations should handle version negotiation and maintain backward compatibility.
