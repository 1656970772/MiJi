---
source_url: "https://github.com/pamirtuna/gamestudio-subagents"
type: webpage
title: "GitHub - pamirtuna/gamestudio-subagents: Game Studio Sub-Agents"
captured_at: 2026-04-23T11:05:36.195514Z
contributor: "unknown"
---

# Game Studio Sub-Agents

Source: https://github.com/pamirtuna/gamestudio-subagents

---

## Summary

Game Studio Sub-Agents presents itself as an AI-powered game development team that runs in the terminal. It organizes multiple specialized agents to support game design, programming, art, QA, market analysis, and data science across different development phases.

## Key Features

- 12 specialized agents covering design, art, programming, QA, market analysis, and data science
- data-driven development with market analysis and analytics from the beginning
- multiple development modes: design-only, prototype, and full development
- automatic project structure generation
- multi-engine support for Godot, Unity, Unreal, and custom engines
- cross-platform orientation for PC, mobile, console, web, and VR/AR
- milestone tracking, market intelligence, and project-specific development rules

## Quick Start

Prerequisites listed in the README include:

- Git
- Python 3.8+
- Node.js
- Claude Code CLI

The documented setup flow is:

1. Clone the repository.
2. Optionally create a virtual environment.
3. Run `python scripts/init_project.py`.
4. Start development through Claude Code prompts.
5. Use `python scripts/project_manager.py` to manage projects.

## AI Team Structure

The repository defines a multi-role team that includes:

- Master Orchestrator
- Producer Agent
- Market Analyst
- Data Scientist
- Sr Game Designer
- Mid Game Designer
- Mechanics Developer
- Game Feel Developer
- Sr Game Artist
- Technical Artist
- UI/UX Agent
- QA Agent

## Development Workflow

The README describes a staged workflow:

1. Market validation
2. Team assembly
3. Parallel development
4. Continuous optimization
5. Launch and post-launch monitoring

The system workflow centers on:

- market analysis before commitment
- producer-led coordination
- analytics and telemetry planning
- iterative optimization based on collected data

## Development Modes

- Design Mode: explore and validate ideas without committing to full production
- Prototype Mode: validate core gameplay quickly
- Development Mode: run the full production pipeline with all agents

## Engine-Specific Positioning

The project states that agents are customized for engine choice:

- Godot agents emphasize GDScript, node architecture, signals, and scene composition
- Unity agents emphasize C#, component architecture, Scriptable Objects, UGUI/UI Toolkit, and build pipelines
- Unreal agents emphasize Blueprint + C++, Gameplay Framework patterns, Material Editor, UMG, and platform optimization

## Project Structure

The generated project structures differ by engine, but each project includes:

- `agents/`
- `project-config.json`
- `documentation/`
- `source/`
- `qa/`
- `builds/`

## Notable Signals For Graph Extraction

- uses Claude Code CLI as the main execution surface
- uses a master-orchestrator / producer style coordination model
- targets multi-agent game development workflows
- explicitly spans Godot, Unity, and Unreal ecosystems
