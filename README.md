# Roblox Studio Dev Skill

All-in-one AI agent skill for Roblox game development. Contains the complete MCP Bridge remote workflow + full Luau/Roblox knowledge base.

## What This Skill Does

When installed, any AI agent working with a Roblox MCP Bridge workspace will automatically:

1. **Test the connection** to Roblox Studio
2. **Pull all scripts** before touching any code
3. **Work locally** in `roblox-src/`
4. **Push changes** back to Studio
5. **Follow Roblox best practices** (security, performance, patterns)

## Install (Global – works in every workspace)

### Windows (PowerShell)

```powershell
$skillDir = "$env:USERPROFILE\.agents\skills\roblox-studio-dev"
New-Item -ItemType Directory -Path $skillDir -Force
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/KS55aa/testskill/main/SKILL.md" -OutFile "$skillDir\SKILL.md"
```

### Mac / Linux

```bash
mkdir -p ~/.agents/skills/roblox-studio-dev
curl -o ~/.agents/skills/roblox-studio-dev/SKILL.md \
  https://raw.githubusercontent.com/KS55aa/testskill/main/SKILL.md
```

### Manual

1. Clone this repo: `git clone https://github.com/KS55aa/testskill.git`
2. Copy `SKILL.md` to `~/.agents/skills/roblox-studio-dev/SKILL.md`

## Verify Installation

Ask your AI agent: "Welche Roblox Skills hast du?" – It should describe the MCP Bridge workflow and Luau knowledge.

## What's Inside

| Section | Content |
|---------|---------|
| MCP Bridge Workflow | Connection test, Pull/Read/Edit/Push cycle, creating new scripts, API reference |
| Luau Language | Types, strict mode, OOP, metatables, task library |
| Architecture | Client-server model, containers, script types |
| Networking | RemoteEvents, RemoteFunctions, validation rules |
| DataStores | CRUD, session locking, auto-save, data manager pattern |
| UI System | ScreenGui, layouts, code-generated UI, TweenService |
| Physics | Constraints, network ownership, collision groups |
| Security | Anti-cheat golden rules, validation pattern |
| Performance | Object pooling, parallel Luau, optimization tips |
| Patterns | Round system, matchmaking queue, debounce, signals |
| MarketplaceService | DevProducts, GamePasses, ProcessReceipt |
| Cross-Server | MessagingService, MemoryStoreService |

## Requirements

- [Roblox MCP Bridge](https://github.com/KS55aa/testskill) (Plugin + Cloud API)
- Roblox Studio with HTTP Requests enabled
- PowerShell (for pull/push scripts)

## License

MIT
