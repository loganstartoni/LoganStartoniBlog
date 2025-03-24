---
title: Preferred Python Pre-commit Config
summary: Example Pre-commit Config

authors:
    - Logan Startoni
date: 2025-03-23
---
# Overview 
Below is my example pre-commit yaml configuration. It does some general file parsing, then it checks python files for the configured ruff linting and formatting rules.

```yaml
# See https://pre-commit.com for more information
# See https://pre-commit.com/hooks.html for more hooks
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v3.2.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
    -   id: check-added-large-files
  - repo: https://github.com/astral-sh/ruff-pre-commit
    # Ruff version.
    rev: v0.11.2
    hooks:
      - id: ruff
        args: [ "--fix", "src"]
      - id: ruff-format
        args: [ src ]

```