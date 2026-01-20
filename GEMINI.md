# Everything Claude Code (Python Edition)

## Project Overview
This repository is a fork of "Everything Claude Code", originally by Affaan Mustafa. It is being adapted to provide a comprehensive set of configurations (Agents, Skills, Rules, Commands) optimized for a **Python + htmx** development stack.

The goal is to provide a "battle-tested" setup for AI assistants to help with Python development, specifically using FastAPI, SQLAlchemy, and htmx.

## Directory Structure

| Directory | Description |
|-----------|-------------|
| `agents/` | Specialized sub-agent definitions (e.g., `planner`, `code-reviewer`). |
| `commands/` | Slash commands for quick actions (e.g., `/tdd`, `/plan`). |
| `rules/` | Guidelines that the AI must always follow (Security, Style, Testing). |
| `skills/` | Workflow definitions and domain knowledge (e.g., TDD workflow). |
| `hooks/` | Automation hooks (Pre/Post tool use). |
| `mcp-configs/` | Model Context Protocol server configurations. |
| `examples/` | Example configurations. |

## Target Technology Stack
The configurations in this repository are being migrated to support:
*   **Language:** Python
*   **Web Framework:** FastAPI
*   **ORM:** SQLAlchemy
*   **Templating:** Jinja2
*   **Validation:** Pydantic
*   **Frontend:** htmx
*   **Testing:** pytest
*   **Linting/Formatting:** Ruff
*   **Type Checking:** mypy

## Active Development Tasks (Migration)
The primary focus currently is migrating existing TypeScript/React conventions to Python:
*   [ ] **Agents:** Update `agents/*.md` to reflect Python tools and workflows.
*   [ ] **Rules:** Rewrite `rules/*.md` for Python coding styles (PEP 8, Ruff) and security.
*   [ ] **Commands:** Adapt `/tdd`, `/build-fix` etc., to use `pytest` and Python error analysis.
*   [ ] **Skills:** Replace JS-centric patterns with Python/FastAPI patterns.
*   [ ] **Hooks:** Change triggers from `npm`/`tsc` to `pip`/`mypy`/`ruff`.

## Development Conventions
When working on this repository or using these configs:
*   **Small Files:** Prefer many small, high-cohesion files (200-400 lines) over monolithic ones.
*   **TDD:** Test Driven Development is strictly enforced.
*   **Security:** No hardcoded secrets. Validate all inputs.
*   **Immutability:** Avoid mutating state where possible (functional core).

## Building & Verification
Since this is a configuration repo, "building" implies verifying the Markdown files and JSON configs are valid.
*   **Validation:** Ensure JSON files (`hooks.json`, `mcp-servers.json`) are syntactically correct.
*   **Testing:** Test the commands and agents in a real Python environment (e.g., by creating a temporary Python project and applying these configs).
