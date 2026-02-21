# Orchestrator Terminal v0.1

A local command center for multi-AI discussions. Single HTML file, zero dependencies, fully offline.

> Built for anyone who orchestrates discussions across multiple AI platforms (ChatGPT, Gemini, Claude, etc.) and is tired of juggling browser tabs.

## The Problem

If you use multiple AIs in parallel — getting different perspectives, cross-checking reasoning, running structured debates — you already know the pain:

| Friction | What This Tool Does |
|---|---|
| Alt-tabbing between 4+ browser tabs + notepad | One view, two columns |
| Manually copy-pasting and removing the target AI's own response | "Copy for [AI]" button auto-excludes + arranges context |
| Forgetting which AI you last sent to | "LAST" badge persists on the button |
| Tiny text boxes for long responses | Focus Mode — expand one block, hide the rest |
| Losing everything on browser refresh | localStorage auto-save |
| Risk of losing a session on browser crash | Export/Import JSON session backup |

## Features

- **4 AI Panels** — ChatGPT, Gemini, Z.ai, Claude (tabs + Ctrl+1/2/3/4 shortcuts)
- **4 Response Textareas** — one per AI, contextual (not a single generic input)
- **Rotational Copy Logic** — fixed order (GPT→Gemini→Z.ai→Claude), excludes target AI, skips empty sections, `---` separators
- **Focus Mode** — click an AI header to expand its textarea full-height; click again to minimize
- **Last Copy Indicator** — "LAST" badge on the most recently used Copy button, persists after refresh
- **Instruction Modes** — Anti-Yes Man / Convergence / Custom (appended to clipboard output)
- **Save/Load Session** — download/upload JSON with structure validation before overwrite
- **Auto-save** — localStorage with 2-second debounce + 💾 indicator
- **Responsive** — stacks vertically below 768px

## Quick Start

1. Download `orchestrator-terminal.html`
2. Open in Chrome (not Incognito)
3. Done

No install. No server. No dependencies.

## How the Copy Logic Works

When you click "Copy for [Target AI]", the clipboard output follows this structure:

1. **Your response for Target** (if any — this is what Target hasn't read yet)
2. **Other AIs in rotation order** (starting from the AI after Target): each AI's response + your response for that AI
3. **Instruction mode text** (always last)

Example — **Copy for Gemini** (all fields filled):
```
aku (darma):          ← your response for Gemini
---
respon z.ai:          ← Z.ai (next after Gemini in rotation)
---
aku (darma):          ← your response for Z.ai
---
respon claude:
---
aku (darma):          ← your response for Claude
---
respon chatgpt:
---
aku (darma):          ← your response for ChatGPT
---
[Instruction Mode]
```

Empty sections are automatically skipped — no empty labels, no orphan separators.

## Constraints (by design)

- Single HTML file (CSS + JS inline)
- Zero libraries, frameworks, CDN, backend, or API calls
- Fully offline — open directly from file explorer
- localStorage only — no server-side storage

## Documentation

| File | Contents |
|---|---|
| `ORCHESTRATOR-TERMINAL-v0.1-FINAL-REVISI.md` | Full spec: problem, layout, components, copy logic, acceptance criteria |
| `EXECUTION-PLAN-v0.1-REVISI.md` | Build steps, test checklist, risk mitigations |

## Background

This tool was born from a real workflow — orchestrating structured multi-AI discussions using browser tabs and a notepad. The manual process worked but created significant cognitive overhead: context switching, prompt stitching, losing track of conversation state.

Built with a "fit-for-purpose" philosophy — minimal features that are fully functional, not minimal features that are broken. Designed through a 4-AI collaborative discussion process with a human orchestrator making all final decisions.

## Customization

The clipboard output uses `aku (darma):` as the label for the orchestrator's responses. To change this to your own name, search and replace `aku (darma)` in the HTML file with your preferred label (e.g. `aku (john):` or `me:`).

## License

MIT
