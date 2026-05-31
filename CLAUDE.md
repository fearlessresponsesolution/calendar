# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A fully offline shift scheduling calendar for ~30 team members, delivered as a **single `index.html`** file with no build step, no dependencies, and no external URLs. Opens directly from `file://` in any modern browser.

- Live: https://fearlessresponsesolution.github.io/calendar/
- Remote: `git@github.com:fearlessresponsesolution/calendar.git` (SSH only — no tokens)

## Commands

**Run the app:**
```bash
wslview index.html          # open in Windows browser from WSL
python3 -m http.server 8080 # or serve locally, then visit http://localhost:8080
```

**Run tests:**
```bash
python3 -m http.server 8080
# open http://localhost:8080/tests/index.html — results render in the page, no CLI runner
```

There is no build, lint, or compile step.

## Rules

@.claude/rules/architecture.md
@.claude/rules/data-model.md
@.claude/rules/critical-rules.md
@.claude/rules/code-patterns.md
@.claude/rules/known-issues.md
@.claude/rules/files-and-git.md
@.claude/rules/do-not.md
