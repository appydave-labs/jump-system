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

Install `jq` if not already present:

```bash
command -v jq &>/dev/null || brew install jq
```

---

## Installation

**For AI agents**: Execute each step in order. Steps are self-contained shell commands — run them exactly as written.

### Step 1 — Clone the repo

Check your git protocol first, then clone:

```bash
GIT_PROTOCOL=$(gh config get git_protocol 2>/dev/null || echo "https")

if [[ "$GIT_PROTOCOL" == "ssh" ]]; then
  git clone git@github.com:appydave-labs/jump-system.git ~/dev/labs/jump-system \
    || git clone https://github.com/appydave-labs/jump-system.git ~/dev/labs/jump-system
else
  git clone https://github.com/appydave-labs/jump-system.git ~/dev/labs/jump-system
fi
```

If both fail, force HTTPS explicitly:

```bash
gh config set git_protocol https
gh repo clone appydave-labs/jump-system ~/dev/labs/jump-system
```

### Step 2 — Make the generator executable

```bash
chmod +x ~/dev/labs/jump-system/generate-aliases.sh
```

### Step 3 — Customise your locations

Edit `~/dev/labs/jump-system/locations.json`. Add, remove, or change entries to match your folder structure. Each entry:

```json
{
  "key": "short-identifier",
  "path": "~/dev/my-project",
  "jump": "jmy",
  "group": "root",
  "description": "My project folder"
}
```

Keep alias names short and `j`-prefixed. The `group` field organises aliases into labelled sections in the output file.

### Step 4 — Verify the generator output before writing

Always dry-run first:

```bash
cd ~/dev/labs/jump-system && ./generate-aliases.sh --dry-run
```

Confirm aliases appear in the output (look for lines starting with `alias`). If only the header prints and no aliases appear, check `locations.json` is valid JSON:

```bash
jq '.' ~/dev/labs/jump-system/locations.json
```

### Step 5 — Generate the aliases file

```bash
cd ~/dev/labs/jump-system && ./generate-aliases.sh
```

The script will report how many aliases were written. If it reports 0, stop and check `locations.json`.

Verify the file was written correctly:

```bash
grep '^alias' ~/.config/jump/aliases-jump.zsh
```

### Step 6 — Wire the aliases file into your shell

Run this block as-is — it detects oh-my-zsh and handles both cases:

```bash
if [ -d ~/.oh-my-zsh/custom ]; then
  cp ~/.config/jump/aliases-jump.zsh ~/.oh-my-zsh/custom/aliases-jump.zsh
  echo "Installed to oh-my-zsh custom folder."
else
  grep -q 'aliases-jump.zsh' ~/.zshrc \
    || echo '[ -f ~/.config/jump/aliases-jump.zsh ] && source ~/.config/jump/aliases-jump.zsh' >> ~/.zshrc
  echo "Added source line to ~/.zshrc."
fi
```

### Step 7 — Verify the installation (non-interactive)

This works without opening a new terminal:

```bash
source ~/.config/jump/aliases-jump.zsh && type jd
```

Expected output: `jd is an alias for cd ~/dev`

If `jd` isn't defined, check that `locations.json` contains an entry with `"jump": "jd"` and re-run Step 5.

---

## Updating your locations

1. Edit `locations.json`
2. Run `./generate-aliases.sh --dry-run` to verify
3. Run `./generate-aliases.sh`
4. Run `source ~/.config/jump/aliases-jump.zsh` (or open a new terminal)

---

## Troubleshooting

**Clone fails with SSH error**
Your SSH key isn't configured for GitHub. Use HTTPS instead:
```bash
git clone https://github.com/appydave-labs/jump-system.git ~/dev/labs/jump-system
```

**Script produces header only, no aliases**
You're on macOS default Bash 3.2 with an older version of this script. Pull the latest version (`git pull`) — the current script uses pure `jq` and is Bash 3.2 safe.

**`jq` not found**
```bash
brew install jq
```

**Alias not found after running the generator**
Source the file manually:
```bash
source ~/.config/jump/aliases-jump.zsh && type jd
```

**oh-my-zsh custom folder doesn't exist**
```bash
mkdir -p ~/.oh-my-zsh/custom
```
Then re-run Step 6.

---

## Future

This is a standalone bash implementation. When the `appydave-tools` Ruby gem becomes available, replace `generate-aliases.sh` with:

```bash
jump generate aliases
```

Your `locations.json` format is compatible — no changes needed.

---

**Part of [appydave-labs](https://github.com/appydave-labs)**
