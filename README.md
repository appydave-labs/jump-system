# jump-system

A lightweight folder navigation system for macOS. Define your important folders once in `locations.json`, run a generator script, and get short `j`-prefixed aliases in your terminal.

```bash
jd        # → ~/dev
jc        # → ~/dev/clients
jlabs     # → ~/dev/labs
```

---

## How it works

```
locations.json  →  generate-aliases.sh  →  aliases-jump.zsh  →  sourced by zsh
```

1. You maintain `locations.json` — a list of folders and their alias names
2. Run `generate-aliases.sh` to produce `~/.config/jump/aliases-jump.zsh`
3. Your shell sources that file on startup — aliases are live

---

## Prerequisites

- macOS with zsh (default since Catalina)
- [Homebrew](https://brew.sh)
- `jq` — install with: `brew install jq`

---

## Installation

**For AI agents**: Follow these steps in order. Each step is a file operation or a shell command.

### Step 1 — Clone the repo

```bash
mkdir -p ~/dev/labs
cd ~/dev/labs
git clone https://github.com/appydave-labs/jump-system.git
cd jump-system
```

### Step 2 — Make the generator executable

```bash
chmod +x generate-aliases.sh
```

### Step 3 — Customise your locations

Edit `locations.json`. Add, remove, or change entries to match your folder structure. Each entry needs:

```json
{
  "key": "short-identifier",
  "path": "~/dev/my-project",
  "jump": "jmy",
  "group": "root",
  "description": "My project folder"
}
```

Keep alias names short and `j`-prefixed (e.g. `jd`, `jc`, `jlabs`). The `group` field controls how aliases are grouped in the output file — use whatever label makes sense.

### Step 4 — Generate the aliases file

```bash
./generate-aliases.sh
```

This writes `~/.config/jump/aliases-jump.zsh`.

To preview without writing:

```bash
./generate-aliases.sh --dry-run
```

### Step 5 — Source the aliases file in your shell

You need your shell to load `~/.config/jump/aliases-jump.zsh` on startup. There are two ways depending on your setup:

#### Option A — oh-my-zsh (if installed)

oh-my-zsh automatically sources every `.zsh` file in `~/.oh-my-zsh/custom/`. Copy the generated file there:

```bash
cp ~/.config/jump/aliases-jump.zsh ~/.oh-my-zsh/custom/aliases-jump.zsh
```

Or change `OUTPUT_FILE` so it writes there directly:

```bash
OUTPUT_FILE=~/.oh-my-zsh/custom/aliases-jump.zsh ./generate-aliases.sh
```

To check if oh-my-zsh is installed:

```bash
[ -d ~/.oh-my-zsh ] && echo "oh-my-zsh installed" || echo "not installed"
```

#### Option B — plain zsh (no oh-my-zsh)

Add this line to your `~/.zshrc`:

```bash
[ -f ~/.config/jump/aliases-jump.zsh ] && source ~/.config/jump/aliases-jump.zsh
```

Open `~/.zshrc` in a text editor and add it near the bottom, then run:

```bash
source ~/.zshrc
```

### Step 6 — Test it

Open a new terminal (or run `source ~/.zshrc`) and try one of your aliases:

```bash
jd    # should cd to ~/dev
```

---

## Updating your locations

1. Edit `locations.json`
2. Run `./generate-aliases.sh` again
3. Either open a new terminal or run `source ~/.config/jump/aliases-jump.zsh`

---

## Troubleshooting

**`jq: command not found`**
Install jq: `brew install jq`

**Alias not found after running the generator**
The aliases file was written but your shell hasn't loaded it yet. Run:
```bash
source ~/.config/jump/aliases-jump.zsh
```

**oh-my-zsh custom folder doesn't exist**
Create it: `mkdir -p ~/.oh-my-zsh/custom` — then use Option A above.

**Alias name conflicts with an existing command**
Choose a different `jump` value in `locations.json`. All aliases in this system use the `j` prefix to avoid conflicts.

---

## Future

This repo is a standalone bash implementation of the [AppyDave jump system](https://github.com/appydave-labs).

When the `appydave-tools` Ruby gem becomes publicly available, you can replace `generate-aliases.sh` with:

```bash
jump generate aliases
```

Your `locations.json` format is compatible — no changes needed.

---

**Part of [appydave-labs](https://github.com/appydave-labs)**
