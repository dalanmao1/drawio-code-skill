# drawio-code-skill

Generate `.drawio` XML files and export to PNG/SVG/PDF/JPG locally using the native draw.io desktop app CLI.

## Overview

A Claude Code skill for creating polished, precise diagrams — architecture, network, UML, ERD, flowcharts, ML/DL model figures, and more. Supports 10,000+ stock/branded shapes, swimlanes, custom geometry, and exported as editable PNG/SVG/PDF.

**Source:** Based on [drawio-skill](https://github.com/Agents365-ai/drawio-skill) v1.14.0 (MIT).

## Structure

```
drawio-code-skill/
├── SKILL.md                          # Main skill instructions
├── data/
│   ├── shape-index.json.gz           # 10,446 official draw.io shapes index
│   ├── SHAPE-INDEX-NOTICE.md         # Attribution notice
│   └── lobe-icons.json               # AI/LLM brand logo manifest (321 brands)
├── scripts/
│   ├── autolayout.py                 # Graphviz-based auto-layout
│   ├── validate.py                   # Structural .drawio linter
│   ├── shapesearch.py                # Search 10k+ official shapes
│   ├── aiicons.py                    # AI/LLM brand logo resolver
│   ├── repair_png.py                 # Fix draw.io -e PNG IEND truncation
│   ├── encode_drawio_url.py          # Browser fallback URL encoder
│   ├── pyimports.py                  # Python import graph extractor
│   ├── jsimports.py                  # JS/TS import graph extractor
│   ├── goimports.py                  # Go import graph extractor
│   ├── rustimports.py                # Rust module-use graph extractor
│   └── pyclasses.py                  # Python class hierarchy extractor
├── references/
│   ├── diagram-types.md              # ERD, UML, Sequence, ML, Flowchart presets
│   ├── shapes.md                     # Shape vocabulary & search guide
│   ├── style-presets.md              # Style preset management
│   ├── style-extraction.md           # Extract styles from existing diagrams
│   ├── autolayout.md                 # Auto-layout usage guide
│   └── troubleshooting.md            # Common mistakes & fixes
└── styles/
    ├── schema.json                   # Preset JSON schema
    └── built-in/
        ├── default.json              # Default color palette
        ├── corporate.json            # Corporate style
        └── handdrawn.json            # Hand-drawn sketch style
```

## Prerequisites

- draw.io desktop app CLI (`drawio` or `draw.io`)
- Optional: Graphviz (`dot`) for auto-layout
- Optional: Vision-enabled model for self-check

## Installation

Install via the Claude Code plugin marketplace.

### 1. Add the marketplace

In a Claude Code session, run:

```
/plugin marketplace add dalanmao1/drawio-code-skill
```

### 2. Install the plugin

```
/plugin install drawio-code-skill@drawio-code-skill
```

The format is `<plugin-name>@<marketplace-name>`. After installation, the `drawio` skill is automatically registered.

### 3. Verify

Start a new Claude Code session and try:

```
Draw a simple client-server architecture diagram
```

The `drawio` skill should auto-trigger, generate a `.drawio` file, and export a PNG.

### Update / Uninstall

```
/plugin update drawio-code-skill
/plugin uninstall drawio-code-skill
```

## Usage

Invoke via Claude Code skill system. The skill auto-activates when users request diagrams, flowcharts, architecture diagrams, ER diagrams, UML, sequence diagrams, network topology, ML/DL model figures, mind maps, or any visualization.

## License

MIT — same as upstream [drawio-skill](https://github.com/Agents365-ai/drawio-skill).