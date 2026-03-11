# Requirements: Jump System as a Claude Code Skill

**Purpose**: Capture what a skill-based implementation of the jump system would look like, why it would be superior to the current Ruby gem + bash script approach, and what requirements it would need to meet.

**Context**: This document was written after running a real-world installation of the bash-based jump system with a new user (Lars). Several failure modes emerged that a skill would have handled better. A future video will demonstrate this approach.

**Created**: 2026-03-11

---

## Background: What We Built and What Went Wrong

The current jump system has two layers:

1. **AppyDave Ruby gem** (`appydave-tools`) — the full system with `jump list`, `jump generate aliases`, `jump add`, etc.
2. **Bash fallback** (`jump-system` lab) — a standalone version for users without the Ruby gem

The bash fallback exposed real problems during Lars's installation:

| Problem | Root cause |
|---------|------------|
| Silent failure — no aliases generated | Bash 3.2 `while read <<<` + `set -e` incompatibility |
| SSH clone failed with no guidance | No protocol detection, no HTTPS fallback |
| oh-my-zsh detection was a manual decision | README said "check this, then do A or B" — agents don't skim |
| No way to verify installation non-interactively | "Open a new terminal" doesn't work for AI agents |
| No way to list aliases from the start | Had to add `jlist` reactively after the session |
| No way to add locations without editing JSON | No guided workflow — raw file edit required |

Every one of these problems either disappears or becomes trivially solvable with a skill.

---

## Why a Skill Would Have Been Better

Skills didn't exist when the Ruby gem was built. If they had, the entire jump system could have been a skill — no Ruby, no bash script, no compatibility matrix, no installation ceremony.

Here's the fundamental difference:

**Current approach**: Ship tooling → user installs tooling → tooling does operations
**Skill approach**: Ship a SKILL.md → AI reads it → AI does operations conversationally

The skill IS the tool. The AI is the runtime.

---

## What the Skill Would Do

### Core operations

| User says | Skill action |
|-----------|-------------|
| "list my jumps" / `jlist` | Read `locations.json`, format as table, print |
| "add a jump to ~/dev/myproject called jmp" | Append entry to `locations.json`, regenerate aliases |
| "remove the jmp alias" | Delete entry from `locations.json`, regenerate aliases |
| "generate my aliases" | Read `locations.json`, write `aliases-jump.zsh`, wire into shell |
| "install jump system" | Full guided flow — detect shell, generate, wire, verify |
| "show me jumps for client work" | Filter by group, show subset |
| "what's the jump for my flivideo folder?" | Search by path or description |

### Installation flow (what the skill would do that the README couldn't)

The skill reads the machine state and acts — it doesn't present options for the human to choose:

1. Check if `jq` is installed → install via brew if not
2. Detect shell setup (oh-my-zsh present? → use custom folder; otherwise → append to `.zshrc`)
3. Detect git protocol → clone via HTTPS if SSH isn't configured
4. Generate aliases
5. Write to correct location
6. Source the file in the current shell context
7. Verify by running `type jd` (or first alias in locations.json)
8. Report result

No branching instructions for the human to navigate. No "Option A / Option B".

---

## Requirements

### REQ-01: Skill file structure

```
jump-system/
├── .claude/
│   └── skills/
│       └── jump/
│           ├── SKILL.md          # main skill — loaded by Claude
│           └── locations.json    # starter template
```

Or as a personal skill at `~/.claude/skills/jump/` for the individual user.

The skill lives alongside the data. No separate installation of a runtime.

### REQ-02: Trigger phrases

The skill description must include trigger words so Claude activates it automatically:

- "list my jumps", "show jump aliases", "jlist"
- "add a jump", "add location", "register folder"
- "remove jump", "delete location"
- "generate jump aliases", "rebuild aliases"
- "install jump system", "set up jump"
- "where is my [project name]"

### REQ-03: locations.json as the single source of truth

Same schema as the current system — no migration needed. The skill reads and writes `locations.json` directly using Claude's file tools. No bash, no jq dependency.

```json
{
  "locations": [
    {
      "key": "dev",
      "path": "~/dev",
      "jump": "jd",
      "group": "root",
      "description": "Development root"
    }
  ]
}
```

Config location: `~/.config/jump/locations.json` (same as current system).

### REQ-04: Alias generation without bash

The skill generates `aliases-jump.zsh` by reading the JSON and writing the file directly — no bash script, no jq, no compatibility concerns. Claude writes the file using its Write tool.

Output format is identical to the current system — compatible with `source` and oh-my-zsh.

### REQ-05: Shell wiring — single conditional, no user decision

The skill detects the environment and acts:

```
if ~/.oh-my-zsh/custom exists:
    write aliases to ~/.oh-my-zsh/custom/aliases-jump.zsh
else:
    append source line to ~/.zshrc if not already present
```

The user is never asked to choose.

### REQ-06: Non-interactive verification

After installation, the skill verifies by sourcing the aliases file and checking that at least one alias resolves. It reports success or failure with a specific error message, not "open a new terminal and try".

### REQ-07: Add location workflow — guided, not raw JSON edit

When the user says "add a jump to ~/dev/myproject":

1. Skill proposes a key, jump alias, group, and description based on the path
2. User confirms or adjusts
3. Skill writes to `locations.json`
4. Skill regenerates `aliases-jump.zsh`
5. Skill sources the updated file

No JSON editing. No regeneration ceremony.

### REQ-08: jlist — always available

`jlist` is not a bash alias that needs to be generated first. With the skill, "list my jumps" works before any installation — the skill just reads `locations.json` and formats the output directly in the conversation.

This fixes the chicken-and-egg problem: you couldn't see your aliases before the system was installed.

### REQ-09: Portable — no Ruby, no jq, no bash version dependency

The skill runs on any machine with Claude Code installed. The only dependencies:
- Claude Code (the user already has it)
- `~/.config/jump/locations.json` (the skill creates this on first run)

### REQ-10: Compatibility with future Ruby gem

When `appydave-tools` becomes available, the skill can detect it and delegate:

```
if `jump` command available:
    run: jump generate aliases
else:
    use built-in file generation
```

Same `locations.json` format — no migration.

---

## Skill SKILL.md outline

```markdown
---
name: jump
description: >
  Manage jump folder aliases. Use when user says "list my jumps", "add a jump",
  "jlist", "where is [project]", "install jump system", "generate jump aliases",
  "add location", "remove jump", "set up jump".
version: 1.0
---

# Jump System Skill

## What this skill manages
...

## Locations file
Path: ~/.config/jump/locations.json
Schema: ...

## Operations

### List locations
...

### Add location
...

### Remove location
...

### Generate aliases
...

### Install (full flow)
...

### Verify installation
...

## Shell wiring logic
...

## Compatibility
...
```

---

## What this would look like in practice

Lars's entire installation session — which took multiple debug iterations and ~10 attempts to get right — would have been:

> "Install the jump system for me"

The skill would have:
1. Detected HTTPS was needed (no SSH key)
2. Cloned the repo
3. Detected oh-my-zsh
4. Generated and wired the aliases
5. Verified `jd` resolved
6. Printed the alias table

One prompt. Done.

---

## What to build for the video

1. Extract the skill from this concept into `~/.claude/skills/jump/SKILL.md`
2. Demo: fresh machine, one prompt installs the whole system
3. Demo: "add a jump to my new project folder" — guided workflow
4. Demo: "jlist" before and after adding
5. Compare: bash script installation (10 iterations) vs skill installation (1 prompt)
6. Discuss: when skills replace CLI tools, and when they don't

---

## Related

- Current lab: `~/dev/labs/jump-system/`
- Ruby gem source: `~/dev/ad/appydave-tools/lib/appydave/tools/jump/`
- Skills system reference: `~/dev/ad/brains/anthropic-claude/` (skills documentation)
- Skill creation guide: `~/.claude/skills/skill-creator/`
