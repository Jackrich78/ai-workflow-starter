# FEAT-005: n8n MCP Integration

**Status:** Implementation Complete
**Created:** 2025-12-08
**Type:** External Tooling Integration

## Problem Statement

We need Claude to understand n8n workflow automation and be able to assist with designing, creating, and managing n8n workflows. The n8n-mcp project provides a Model Context Protocol server that gives Claude access to n8n node documentation, workflow templates, and optional live workflow management.

## Solution

Integrate the n8n-mcp Docker image with Claude Code using an **on-demand add/remove approach**. This keeps the MCP available when needed without consuming resources in every session.

## Goals

1. Claude can answer questions about n8n nodes and their configuration
2. Claude can reference 2,700+ workflow templates for design patterns
3. Claude can create, update, and execute workflows on our n8n instance
4. Configuration is documented for easy on-demand activation
5. MCP Configuration SOP created for future integrations

## Usage Model

**On-demand activation** rather than always-on:
- Add the MCP when starting n8n work
- Remove when done to free resources
- Docker image stays cached for quick re-activation

This approach:
- Avoids ~150-200MB RAM usage in unrelated sessions
- Doesn't require Docker Desktop running for non-n8n work
- Keeps API key out of config when not in use

## Scope

**In Scope:**
- Docker image installation (pulled, cached locally)
- Claude Code on-demand configuration (add/remove via CLI)
- Claude Desktop configuration (optional, via JSON)
- MCP Configuration SOP creation
- Connection to hosted n8n at `https://sherpasdurden.app.n8n.cloud`
- Risk assessment and security documentation

**Out of Scope:**
- Running n8n locally in Docker (using hosted instance)
- Forking/cloning the n8n-mcp repository
- Custom modifications to n8n-mcp
- Always-on MCP configuration

## Technical Approach

- **Transport:** STDIO mode (required for Claude integrations)
- **Container:** Pre-built Docker image `ghcr.io/czlonkowski/n8n-mcp:latest`
- **API Connection:** n8n Cloud instance with API key authentication
- **Lifecycle:** Add when needed, remove when done

## Related Documentation

- [Installation Notes](./notes.md) - Commands, risk assessment, troubleshooting
- [MCP Configuration SOP](../../sop/mcp-configuration.md) - General MCP setup guidance
