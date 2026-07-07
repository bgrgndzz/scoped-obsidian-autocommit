# scoped-obsidian-autocommit

Auto-commit one Obsidian subtree to one Git remote without exposing the rest of
your vault.

This is for the common case where your full Obsidian vault contains private
notes, but one folder should become a shared Git-backed knowledge base.

## How it works

The tool never runs `git add` in your full vault. Instead it:

1. reads a configured source folder,
2. mirrors only that folder into a separate local Git repo,
3. commits changes in the mirror repo,
4. optionally pushes the mirror repo to GitHub.

```text
Private Obsidian Vault/
  AI Research/       -> mirrored -> ~/Developer/ai-research-zettelkasten -> GitHub
  Private Journal/   -> ignored
  People/            -> ignored
  .obsidian/         -> ignored by default
```

## Install

```bash
git clone https://github.com/bgrgndzz/scoped-obsidian-autocommit.git
cd scoped-obsidian-autocommit
chmod +x bin/scoped-obsidian-autocommit bin/install-launch-agent
```

Or run it directly from this checkout.

## Configure

Copy the example config:

```bash
mkdir -p ~/.config/scoped-obsidian-autocommit
cp examples/ai-research.example.env ~/.config/scoped-obsidian-autocommit/ai-research.env
```

Edit the config:

```bash
SOURCE_PATH="/Users/you/Documents/Obsidian Vault/AI Research"
MIRROR_REPO="/Users/you/Developer/ai-research-zettelkasten"
REMOTE_URL="git@github.com:your-org/ai-research-zettelkasten.git"
BRANCH="main"
MESSAGE_PREFIX="Auto-commit AI research zettelkasten"
PUSH=1
```

Config files are trusted shell snippets. Keep local configs out of Git if they
include private paths or remotes.

## Dry run

Always dry-run first:

```bash
bin/scoped-obsidian-autocommit \
  --config ~/.config/scoped-obsidian-autocommit/ai-research.env \
  --dry-run
```

The dry run uses `rsync -n`, so it shows what would be mirrored without changing
the mirror repo.

## Run once

```bash
bin/scoped-obsidian-autocommit \
  --config ~/.config/scoped-obsidian-autocommit/ai-research.env
```

Use `--no-push` if you want local commits only:

```bash
bin/scoped-obsidian-autocommit \
  --config ~/.config/scoped-obsidian-autocommit/ai-research.env \
  --no-push
```

## Install a macOS LaunchAgent

```bash
bin/install-launch-agent \
  --label com.bourne.ai-research-autocommit \
  --config ~/.config/scoped-obsidian-autocommit/ai-research.env \
  --interval 900
```

This runs every 15 minutes and logs to:

```text
/tmp/com.bourne.ai-research-autocommit.log
```

To unload it:

```bash
launchctl unload ~/Library/LaunchAgents/com.bourne.ai-research-autocommit.plist
```

## Safety defaults

- refuses to publish your home directory or filesystem root
- refuses to use the same path for source and mirror
- refuses to put the mirror repo inside the source path
- refuses to publish a folder containing `.obsidian` unless
  `--allow-obsidian-root` is passed
- excludes `.git/`, `.obsidian/`, `.trash/`, `.DS_Store`, and `Thumbs.db`
- supports repeatable `--exclude` patterns

## Why not just Git in the full vault?

Because `git add -A` at the vault root is too easy to get wrong when the vault
contains private notes. A separate mirror repo makes the privacy boundary a
filesystem boundary instead of a human habit.
