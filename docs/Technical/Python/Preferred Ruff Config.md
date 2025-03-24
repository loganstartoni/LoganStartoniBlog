---
title: Preferred ruff Config
summary: Example ruff Config

authors:
    - Logan Startoni
date: 2025-03-23
---
# Overview
Below is the ruff config that I like to use as a starting point in my pyproject.toml files.

```toml
[tool.ruff]
# Rules that are available to use: https://beta.ruff.rs/docs/rules/
lint.select = [
    "E", "W", # PyCodestyle
    "F", # PyFlakes
    "D", # PyDocstyle
    "PT", # Flake 8 - pytest style
    "N", # Flake 8 - Naming
    "ICN", # Flake 8 - Import Conventions
    "PTH", # Flake 8 - Use Pathlib
    "TD", # Flake 8 - TO DO Style
    "T20", # Flake 8 - No Print
    "SLF", # Flake 8 - Protect Private Methods
    "SIM", # Flake 8 - Simplify the Code
    "ISC", # Flake 8 - Sting Concatenation
    "COM", # Flake 8 - Comma Conventions
    "RSE", # Flake 8 - Raise Patterns
    "C4", # Flake 8 - Conprehensions Patterns
    "FBT", # Flake 8 - Boolean Patterns
]
lint.ignore = [
    "COM812", # Trailing comma missing - conflicts with formatting
    "D100",   # Missing docstring in public module - Generally the code written is for me, so I don't require it.
    "D104",   # Missing docstring in public package - Generally the code written is for me, so I don't require it.
    "D203",   # 1 blank line required before class docstring - conflicts with ruff formatting
    "D205",   # 1 blank line required between summary line and description
    "D212",   # Multi-line docstring summary should start at the first line - I like when they start before the """
    "D401",   # First line of docstring should be in imperative mood: "{first_line}" - I will set the mood I use.
    "TD002",  # Missing to do author - I don't care who wrote it.
    "TD003",  # Missing to do issue - I don't use an issue tracker for personal repos
]


# Allow autofix for all enabled rules (when `--fix`) is provided.
lint.fixable = ["ALL"]
line-length = 120

```