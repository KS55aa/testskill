# Roblox Studio Dev Skill

AI-Agent-Skill für professionelle Roblox-Spieleentwicklung mit Luau. Enthält Architektur-Patterns, Client-Server-Networking, DataStores, UI-System, Physics, Security, Performance-Optimierung und den kompletten **MCP Bridge Workflow** für Remote-Development.

## Installation

### Globale Installation (alle Projekte)

```bash
# Repo klonen
git clone https://github.com/ksinec45/roblox-studio-dev-skill.git

# SKILL.md in den globalen Skills-Ordner kopieren
# Windows:
copy roblox-studio-dev-skill\SKILL.md %USERPROFILE%\.agents\skills\roblox-studio-dev\SKILL.md

# Mac/Linux:
cp roblox-studio-dev-skill/SKILL.md ~/.agents/skills/roblox-studio-dev/SKILL.md
```

### Oder direkt per curl

```bash
# Windows (PowerShell):
New-Item -ItemType Directory -Path "$env:USERPROFILE\.agents\skills\roblox-studio-dev" -Force
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/ksinec45/roblox-studio-dev-skill/main/SKILL.md" -OutFile "$env:USERPROFILE\.agents\skills\roblox-studio-dev\SKILL.md"

# Mac/Linux:
mkdir -p ~/.agents/skills/roblox-studio-dev
curl -o ~/.agents/skills/roblox-studio-dev/SKILL.md https://raw.githubusercontent.com/ksinec45/roblox-studio-dev-skill/main/SKILL.md
```

## Was der Skill abdeckt

| Bereich | Inhalt |
|---------|--------|
| Architektur | Client-Server Model, Container, RunContext |
| Luau | Types, OOP, Metatables, Parallel Luau |
| Networking | RemoteEvents, RemoteFunctions, Validation |
| DataStores | CRUD, Session Locking, Auto-Save |
| UI | ScreenGui, Layouts, TweenService Animations |
| Physics | Constraints, Network Ownership, Collision Groups |
| Security | Anti-Cheat, Input Validation, Rate Limiting |
| Performance | Object Pooling, Profiling, Streaming |
| MCP Bridge | Pull/Push Workflow, API, lokale Entwicklung |

## MCP Bridge Workflow

Wenn der Skill erkennt, dass `scripts/pull.ps1` und `scripts/push.ps1` im Workspace existieren, erzwingt er diesen Workflow:

```
1. .\scripts\pull.ps1              ← Alle Scripts von Roblox Studio holen
2. Dateien in roblox-src/ lesen    ← Code analysieren und verstehen
3. Lokal editieren                 ← Änderungen in roblox-src/ machen
4. .\scripts\push.ps1              ← Zurück nach Roblox Studio deployen
5. In Studio testen                ← Verifizieren
```

**Regel:** Nie direkt über die API editieren. Immer lokal arbeiten und pushen.

## Für AI-Agenten

Wenn du ein AI-Agent bist und diesen Skill geladen hast:

1. **Prüfe ob `scripts/pull.ps1` existiert** → Falls ja, führe den MCP Bridge Workflow aus
2. **IMMER erst `.\scripts\pull.ps1` ausführen** bevor du Code anfasst
3. **Lies die gepullten Dateien** in `roblox-src/` um den aktuellen Stand zu verstehen
4. **Editiere NUR lokale Dateien** in `roblox-src/`
5. **Pushe mit `.\scripts\push.ps1`** nach jeder Änderung

## Voraussetzungen

- [Roblox MCP Bridge](https://github.com/ksinec45/roblox-mcp-bridge) Setup (Plugin + Server)
- Roblox Studio mit aktivierten HTTP Requests
- PowerShell (für die Push/Pull Scripts)

## Lizenz

MIT
