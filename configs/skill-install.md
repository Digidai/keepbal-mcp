# Install the KeepBal Skill

Use the canonical [hosted SKILL.md](https://keepbal.com/SKILL.md). The Bash example below downloads it without authentication and adds `name` and `description` metadata when the hosted file has no YAML frontmatter.

Choose the local destination for your client:

| Client | `keepbal_skill_dir` |
| --- | --- |
| Claude Code | `$HOME/.claude/skills/keepbal` |
| Codex | `$HOME/.agents/skills/keepbal` |

Run this locally, changing `keepbal_skill_dir` for Codex. It creates or replaces that client's local KeepBal Skill file:

```bash
(
  set -eu
  keepbal_skill_dir="$HOME/.claude/skills/keepbal"
  keepbal_skill_source="$(mktemp)"
  trap 'rm -f "$keepbal_skill_source"' EXIT

  curl --fail --silent --show-error --location https://keepbal.com/SKILL.md \
    --output "$keepbal_skill_source"
  mkdir -p "$keepbal_skill_dir"
  {
    if [ "$(head -n 1 "$keepbal_skill_source")" != '---' ]; then
      cat <<'YAML'
---
name: keepbal
description: Search, inspect, run, and check tools through KeepBal with a prepaid wallet.
---

YAML
    fi
    cat "$keepbal_skill_source"
  } > "$keepbal_skill_dir/SKILL.md"
)
```

Reload your client's skills or start a new session. Rerun the download to refresh the guide; tool availability and pricing can change. The Skill provides usage instructions. Set up MCP and `KEEPBAL_API_KEY` separately as described in the [connection guide](../README.md).

Client references: [Claude Code skills](https://code.claude.com/docs/en/skills) and [Codex skills](https://developers.openai.com/codex/skills).
