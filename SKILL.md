---
name: roblox-studio-dev
description: Enterprise-level Roblox game development with Luau. Covers architecture, client-server networking, DataStores, UI, physics, animation, OOP patterns, optimization, and security. Use when building Roblox games, writing Luau scripts, designing game systems, or working with Roblox Studio APIs. Triggers on Roblox, Luau, game development, RemoteEvent, DataStore, TweenService, Roblox Studio.
---

# Roblox Studio Enterprise Development

## Architecture Overview

Roblox uses a client-server model. The server (DataModel) is authoritative. Services replicate to clients automatically (Workspace, ReplicatedStorage). Clients cannot directly modify server state.

### Script Types
- **Script** (ServerScript): Runs on server. RunContext = Server.
- **LocalScript**: Runs on client only. Access to UserInputService, camera, local player.
- **ModuleScript**: Shared code. Returns a table. Executed once per environment via `require()`.

### Container Purposes
| Container | Side | Purpose |
|-----------|------|---------|
| ServerScriptService | Server | Server scripts (invisible to client) |
| ServerStorage | Server | Assets/data hidden from client |
| ReplicatedStorage | Both | Shared modules, RemoteEvents, assets |
| ReplicatedFirst | Client | Loading screen scripts (run first) |
| StarterGui | Client | UI templates cloned to PlayerGui |
| StarterPlayerScripts | Client | Client scripts per player |
| StarterCharacterScripts | Client | Scripts inside character model |
| Workspace | Both | 3D world, parts, models |

### RunContext (Modern approach, replaces LocalScript)
```lua
-- Script with RunContext = Client replaces LocalScript
-- Script with RunContext = Server replaces legacy Script placement rules
```

## Luau Language

### Types & Strict Mode
```lua
--!strict
local count: number = 0
local name: string = "Player"
local active: boolean = true
local data: {[string]: number} = {}
type PlayerData = { coins: number, level: number, inventory: {string} }
```

### OOP Pattern (Metatable-based)
```lua
local MyClass = {}
MyClass.__index = MyClass

function MyClass.new(value: number)
    local self = setmetatable({}, MyClass)
    self.value = value
    return self
end

function MyClass:getValue(): number
    return self.value
end

function MyClass:destroy()
    setmetatable(self, nil)
    table.clear(self)
end
```

### Inheritance
```lua
local Base = {}
Base.__index = Base
function Base.new() return setmetatable({}, Base) end
function Base:method() return "base" end

local Child = setmetatable({}, {__index = Base})
Child.__index = Child
function Child.new()
    local self = Base.new()
    return setmetatable(self, Child)
end
function Child:method() return "child" end
```

### Metatables Key Metamethods
| Metamethod | Trigger |
|-----------|---------|
| `__index` | Accessing nil key |
| `__newindex` | Setting nil key |
| `__call` | Calling table as function |
| `__add/__sub/__mul/__div` | Arithmetic |
| `__tostring` | tostring() |
| `__len` | # operator |
| `__eq/__lt/__le` | Comparison |
| `__iter` | Generalized iteration |

Use `rawset()`/`rawget()` to bypass metamethods when needed.

## Client-Server Communication

### RemoteEvent (Fire-and-forget)
```lua
-- ReplicatedStorage/RemoteEvent
-- SERVER:
remote.OnServerEvent:Connect(function(player, data)
    -- validate ALL client input here
end)
remote:FireClient(player, responseData)
remote:FireAllClients(broadcastData)

-- CLIENT:
remote:FireServer(requestData)
remote.OnClientEvent:Connect(function(data) end)
```

### RemoteFunction (Request-Response)
```lua
-- SERVER:
remoteFunc.OnServerInvoke = function(player, args)
    return result
end

-- CLIENT:
local result = remoteFunc:InvokeServer(args)
```

### Critical Rules
- NEVER trust client data. Validate everything server-side.
- Arguments are copied (not referenced). Tables lose metatables.
- `nil` replaces non-replicable types (functions, userdata, threads).
- Mixed tables (array + dict) may not replicate correctly.
- RemoteFunction:InvokeClient is dangerous (client can hang server). Avoid it.
- Rate-limit remote calls to prevent exploitation.

## DataStoreService

```lua
local DataStoreService = game:GetService("DataStoreService")
local store = DataStoreService:GetDataStore("PlayerData")

-- ALWAYS pcall DataStore operations
local success, data = pcall(function()
    return store:GetAsync("Player_" .. player.UserId)
end)

-- UpdateAsync for atomic read-modify-write (preferred over SetAsync)
pcall(function()
    store:UpdateAsync("Player_" .. player.UserId, function(old)
        old = old or {coins = 0, level = 1}
        old.coins += amount
        return old
    end)
end)
```

### Best Practices
- Use UpdateAsync over SetAsync (prevents race conditions)
- Implement session locking to prevent data duplication
- Auto-save every 30-60 seconds + save on PlayerRemoving + BindToClose
- Budget: 60 + numPlayers×10 GetAsync/min, 60 + numPlayers×10 SetAsync/min
- Key max 50 chars, Value max 4MB
- Use OrderedDataStore for leaderboards

### Session Lock Pattern
```lua
local LOCK_KEY = "lock_"
function acquireLock(store, key, serverId)
    local success = pcall(function()
        store:UpdateAsync(LOCK_KEY .. key, function(old)
            if old and old.serverId ~= serverId and os.time() - old.time < 300 then
                return nil -- abort, locked by another server
            end
            return {serverId = serverId, time = os.time()}
        end)
    end)
    return success
end
```

## TweenService

```lua
local TweenService = game:GetService("TweenService")

local tweenInfo = TweenInfo.new(
    1,                          -- duration
    Enum.EasingStyle.Quad,      -- easing style
    Enum.EasingDirection.Out,   -- easing direction
    0,                          -- repeat count (-1 = infinite)
    false,                      -- reverses
    0                           -- delay
)

local tween = TweenService:Create(part, tweenInfo, {
    Position = Vector3.new(0, 10, 0),
    Transparency = 0.5
})
tween:Play()
tween.Completed:Connect(function(status) end)
```

### EasingStyles
Linear, Sine, Quad, Cubic, Quart, Quint, Exponential, Circular, Back, Bounce, Elastic

### Tween Sequencing
```lua
local tween1 = TweenService:Create(obj, info, {Position = target1})
local tween2 = TweenService:Create(obj, info, {Rotation = target2})
tween1:Play()
tween1.Completed:Connect(function() tween2:Play() end)
```

## UI System

### Hierarchy
- ScreenGui → Frame/TextLabel/TextButton/ImageLabel/ImageButton/TextBox
- SurfaceGui (attached to Part face)
- BillboardGui (always faces camera)

### Layout Objects
- UIListLayout: Arranges children in list
- UIGridLayout: Grid arrangement
- UIPageLayout: Swipeable pages
- UITableLayout: Table rows/cols
- UIPadding, UIStroke, UIGradient, UICorner, UIAspectRatioConstraint

### Sizing
- UDim2.new(scaleX, offsetX, scaleY, offsetY)
- Scale = percentage of parent (0-1), Offset = pixels
- Always prefer Scale for responsive UI

### Typewriter Effect
```lua
function typeWrite(label, text, delay)
    label.MaxVisibleGraphemes = 0
    label.Text = text
    for i = 1, utf8.len(text) do
        label.MaxVisibleGraphemes = i
        task.wait(delay)
    end
end
```

## Physics & Constraints

### Modern Constraints (use these, not deprecated Body objects)
| Constraint | Purpose |
|-----------|---------|
| AlignPosition | Move part to target position |
| AlignOrientation | Rotate part to target orientation |
| LinearVelocity | Constant velocity |
| AngularVelocity | Constant rotation |
| VectorForce | Apply force |
| Torque | Apply torque |
| BallSocketConstraint | Joint (ragdoll) |
| HingeConstraint | Door/wheel |
| SpringConstraint | Spring physics |
| RopeConstraint | Rope |
| WeldConstraint | Rigid connection |

### Network Ownership
```lua
-- Server decides which client simulates physics
part:SetNetworkOwner(player) -- player simulates this part
part:SetNetworkOwner(nil)    -- server simulates
part:SetNetworkOwnershipAuto() -- engine decides
```

### Collision Groups
```lua
local PhysicsService = game:GetService("PhysicsService")
PhysicsService:RegisterCollisionGroup("Players")
PhysicsService:RegisterCollisionGroup("Projectiles")
PhysicsService:CollisionGroupSetCollidable("Players", "Projectiles", false)
part.CollisionGroup = "Players"
```

## Services Reference

| Service | Purpose |
|---------|---------|
| Players | Player management, PlayerAdded/Removing |
| RunService | Heartbeat, RenderStepped, Stepped |
| UserInputService | Keyboard/mouse/touch/gamepad input |
| TweenService | Property animation |
| DataStoreService | Persistent data |
| ReplicatedStorage | Shared assets/modules/remotes |
| ServerStorage | Server-only assets |
| Workspace | 3D world |
| Lighting | Environment lighting, atmosphere |
| SoundService | Audio |
| MarketplaceService | Purchases, DevProducts, GamePasses |
| CollectionService | Tag-based instance management |
| PathfindingService | NPC navigation |
| PhysicsService | Collision groups |
| HttpService | External HTTP (JSON encode/decode) |
| ContextActionService | Mobile/gamepad action binding |
| Chat | Chat system |
| Teams | Team management |
| BadgeService | Award badges |
| TeleportService | Place teleportation |
| MessagingService | Cross-server communication |
| MemoryStoreService | Temporary shared data (matchmaking) |

## Performance & Optimization

### Critical Rules
- Avoid creating/destroying instances in loops (use object pooling)
- Use task.wait() not wait() (more precise)
- Minimize RemoteEvent traffic (batch updates)
- Use CollectionService tags instead of iterating all descendants
- Streaming: Enable InstanceStreaming for large maps
- Avoid :GetDescendants() in hot paths
- Use Parallel Luau (Actor) for CPU-heavy work

### Object Pooling
```lua
local pool = {}
function getFromPool()
    local obj = table.remove(pool)
    if not obj then
        obj = createNewObject()
    end
    obj.Parent = workspace
    return obj
end
function returnToPool(obj)
    obj.Parent = nil
    table.insert(pool, obj)
end
```

### Custom Profiling
```lua
debug.profilebegin("MyExpensiveFunction")
-- code
debug.profileend()
-- View in MicroProfiler (Ctrl+F6)
```

### Parallel Luau
```lua
-- Script inside an Actor instance
local actor = script:GetActor()
task.desynchronize() -- switch to parallel
-- safe parallel work (no shared state mutation)
task.synchronize() -- back to serial for API calls
```

## Security & Anti-Cheat

### Golden Rules
1. Server is ALWAYS authoritative. Never trust the client.
2. Validate ALL RemoteEvent arguments (type, range, frequency).
3. Never store important state client-side only.
4. Sanity-check movement (speed, teleportation detection).
5. Use server-side hit detection for competitive games.
6. Rate-limit remote calls per player.
7. Don't put secret logic in LocalScripts (decompilable).
8. Use ServerStorage/ServerScriptService for sensitive data.

### Validation Pattern
```lua
remote.OnServerEvent:Connect(function(player, action, data)
    if typeof(action) ~= "string" then return end
    if typeof(data) ~= "table" then return end

    local character = player.Character
    if not character then return end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    -- Range check
    if (hrp.Position - targetPos).Magnitude > MAX_RANGE then return end

    -- Rate limit
    if not checkRateLimit(player, action) then return end

    -- Process valid request
end)
```

## Common Game Patterns

### Round System
```lua
local STATUS = {Waiting = 1, Active = 2, Ending = 3}
local currentStatus = STATUS.Waiting

function startRound()
    currentStatus = STATUS.Active
    teleportPlayersToArena()
    startTimer(ROUND_DURATION)
end

function endRound(winner)
    currentStatus = STATUS.Ending
    awardPoints(winner)
    task.wait(INTERMISSION)
    currentStatus = STATUS.Waiting
end
```

### Touched Event (Debounced)
```lua
local debounce = {}
part.Touched:Connect(function(hit)
    local player = game.Players:GetPlayerFromCharacter(hit.Parent)
    if not player then return end
    if debounce[player] then return end
    debounce[player] = true

    -- action here

    task.wait(1)
    debounce[player] = nil
end)
```

### NPC with PathfindingService
```lua
local PathfindingService = game:GetService("PathfindingService")
function moveTo(npc, target)
    local path = PathfindingService:CreatePath({
        AgentRadius = 2,
        AgentHeight = 5,
        AgentCanJump = true,
    })
    path:ComputeAsync(npc.HumanoidRootPart.Position, target)
    if path.Status == Enum.PathStatus.Success then
        for _, wp in path:GetWaypoints() do
            npc.Humanoid:MoveTo(wp.Position)
            if wp.Action == Enum.PathWaypointAction.Jump then
                npc.Humanoid.Jump = true
            end
            npc.Humanoid.MoveToFinished:Wait()
        end
    end
end
```

### Inventory/Data Module Pattern
```lua
local PlayerDataManager = {}
local playerData = {}

function PlayerDataManager.init(player)
    local data = loadFromDataStore(player)
    playerData[player] = data or getDefaultData()
end

function PlayerDataManager.get(player): PlayerData
    return playerData[player]
end

function PlayerDataManager.set(player, key, value)
    playerData[player][key] = value
    markDirty(player)
end

function PlayerDataManager.cleanup(player)
    saveToDataStore(player, playerData[player])
    playerData[player] = nil
end

return PlayerDataManager
```

## Instance Creation & Manipulation

```lua
local part = Instance.new("Part")
part.Size = Vector3.new(4, 1, 4)
part.Position = Vector3.new(0, 5, 0)
part.Anchored = true
part.Material = Enum.Material.Neon
part.BrickColor = BrickColor.new("Bright blue")
part.Parent = workspace -- ALWAYS set Parent last (performance)

-- Clone existing
local clone = template:Clone()
clone.Parent = workspace

-- Destroy
part:Destroy() -- removes from game, GC'd

-- Finding
local child = model:FindFirstChild("PartName")
local descendant = model:FindFirstDescendant("Deep")
local byClass = model:FindFirstChildOfClass("Humanoid")
local byTag = CollectionService:GetTagged("Enemy")

-- WaitForChild (with timeout)
local gui = player:WaitForChild("PlayerGui", 5)
```

## Signals & Events

```lua
-- BindableEvent (same-side communication)
local event = Instance.new("BindableEvent")
event.Event:Connect(function(data) end)
event:Fire(data)

-- Custom signal (no Instance overhead)
local Signal = {}
Signal.__index = Signal
function Signal.new()
    local self = setmetatable({}, Signal)
    self._connections = {}
    return self
end
function Signal:connect(fn)
    table.insert(self._connections, fn)
    return {disconnect = function()
        local idx = table.find(self._connections, fn)
        if idx then table.remove(self._connections, idx) end
    end}
end
function Signal:fire(...)
    for _, fn in self._connections do
        task.spawn(fn, ...)
    end
end
```

## task Library (Modern scheduling)

```lua
task.spawn(fn, ...)       -- immediate new thread
task.defer(fn, ...)       -- next resumption cycle
task.delay(seconds, fn)   -- delayed execution
task.wait(seconds)        -- precise yield
task.cancel(thread)       -- cancel spawned thread
```

## MarketplaceService

```lua
local MPS = game:GetService("MarketplaceService")

-- Developer Products (repeatable purchases)
MPS.ProcessReceipt = function(info)
    local player = game.Players:GetPlayerByUserId(info.PlayerId)
    if not player then return Enum.ProductPurchaseDecision.NotProcessedYet end
    grantProduct(player, info.ProductId)
    return Enum.ProductPurchaseDecision.PurchaseGranted
end

-- GamePass check
local hasPass = MPS:UserOwnsGamePassAsync(player.UserId, PASS_ID)

-- Prompt purchase
MPS:PromptProductPurchase(player, PRODUCT_ID)
MPS:PromptGamePassPurchase(player, PASS_ID)
```

## Cross-Server (MessagingService + MemoryStore)

```lua
-- MessagingService: pub/sub between servers
local MS = game:GetService("MessagingService")
MS:SubscribeAsync("GlobalAnnounce", function(message)
    -- message.Data, message.Sent
end)
MS:PublishAsync("GlobalAnnounce", {text = "Event starting!"})

-- MemoryStoreService: temp shared data (matchmaking, queues)
local MSS = game:GetService("MemoryStoreService")
local queue = MSS:GetQueue("Matchmaking")
queue:AddAsync({playerId = player.UserId, rank = 1500}, 300) -- 300s expiry
local items = queue:ReadAsync(10, false, 5) -- count, allOrNothing, timeout
```

## File Organization (Enterprise)

```
ServerScriptService/
├── Services/          -- Game logic modules (RoundService, CombatService)
├── DataManager.lua    -- Player data handling
├── init.server.lua    -- Bootstrap, initializes all services
ReplicatedStorage/
├── Modules/           -- Shared utilities (Utils, Config, Types)
├── Remotes/           -- Folder of RemoteEvents/Functions
├── Assets/            -- Shared models, effects
StarterPlayerScripts/
├── Controllers/       -- Client controllers (InputController, UIController)
├── init.client.lua    -- Client bootstrap
StarterGui/
├── MainUI/            -- ScreenGui templates
```

## MCP Bridge – Remote Development Workflow

When working with a Roblox project that uses the MCP Bridge (detected by `scripts/push.ps1` or `scripts/pull.ps1` in the workspace), follow this workflow strictly.

### Mandatory Startup Sequence

**BEFORE writing or editing any Luau code, ALWAYS execute these steps in order:**

1. **Pull all scripts from Roblox Studio** → Run `.\scripts\pull.ps1`
2. **Read the pulled files** in `roblox-src/` to understand current state
3. **Only then** start making changes to the local files
4. **After changes** → Run `.\scripts\push.ps1` to deploy

Never skip the pull step. The source of truth is always Roblox Studio until pulled locally.

### Directory Structure

All game scripts live in `roblox-src/` mirroring the Roblox DataModel hierarchy:

```
roblox-src/
  ServerScriptService/
    GameServer.server.lua
    LeaderboardHandler.server.lua
    ShopHandler.server.lua
  ServerStorage/
    GameModules/
      DataManager.module.lua
      TicTacToeLogic.module.lua
      ConnectFourLogic.module.lua
  StarterGui/
    GameClient.client.lua
  ReplicatedStorage/
    ...
```

### File Naming Convention

| Suffix | Roblox ClassName | Runs On |
|--------|-----------------|---------|
| `.server.lua` | Script | Server |
| `.client.lua` | LocalScript | Client |
| `.module.lua` | ModuleScript | Wherever required |

The folder path maps directly to the Roblox instance path:
- `roblox-src/ServerScriptService/GameServer.server.lua` → `game.ServerScriptService.GameServer`
- `roblox-src/ServerStorage/GameModules/DataManager.module.lua` → `game.ServerStorage.GameModules.DataManager`

### Scripts

**`scripts/pull.ps1`** – Pulls ALL scripts from Roblox Studio into `roblox-src/`
- Clears `roblox-src/` first, then recreates from live game state
- Creates `_tree.json` with full instance tree for reference
- Detects script type from ClassName and applies correct file suffix

**`scripts/push.ps1`** – Pushes ALL `.lua` files from `roblox-src/` to Roblox Studio
- Reads each file, resolves game path from folder structure
- Sends source code via MCP Bridge API (`setScriptSource` action)
- Reports success/failure per file

### API Details

| Field | Value |
|-------|-------|
| Base URL | `https://bridge-cloud.vercel.app/api` |
| Auth Header | `X-API-Key` |
| Endpoints | `/command` (execute actions), `/poll` (get pending), `/result` (send back) |

Actions available via `/command`:
- `getGameTree` – Full instance tree
- `getChildren` – Children of a path
- `getScriptSource` – Read script source
- `setScriptSource` – Write script source (params: path, source)
- `getAllScripts` – All scripts with source and className
- `executeScript` – Run arbitrary Luau in Studio command bar

### Editing Workflow (Step by Step)

```
1. .\scripts\pull.ps1              ← Get latest from Studio
2. Read/analyze roblox-src/ files  ← Understand current code
3. Edit files locally              ← Make changes in roblox-src/
4. .\scripts\push.ps1              ← Deploy to Studio
5. Test in Roblox Studio           ← Verify changes work
6. Repeat 3-5 as needed            ← Iterate
```

### Rules

- NEVER use the API directly to edit scripts. Always edit local files and push.
- NEVER create new scripts via API. Create the `.lua` file locally in the correct folder, then push.
- ALWAYS pull before starting work on an existing project.
- The `roblox-src/` folder IS the local source of truth after pulling.
- If push fails with "Instance not found", the script doesn't exist in Studio yet – create it via `executeScript` action first, then push source.

### Creating New Scripts

To add a new script that doesn't exist in Studio yet:

1. Create the file locally: `roblox-src/ServerScriptService/MyNew.server.lua`
2. Create the instance in Studio first:
```powershell
$body = @{id="create"; action="executeScript"; params=@{source="local s = Instance.new('Script'); s.Name = 'MyNew'; s.Parent = game.ServerScriptService"}} | ConvertTo-Json -Depth 5
Invoke-RestMethod -Uri "$baseUrl" -Method POST -Headers $h -Body ([System.Text.Encoding]::UTF8.GetBytes($body))
```
3. Then push: `.\scripts\push.ps1`

## References

See `references/` directory for detailed docs on specific topics:
- architecture.md - Deep dive on game architecture patterns
- networking.md - Advanced networking, replication, optimization
- data-stores.md - Full DataStore patterns with error handling
- ui-system.md - Complete UI reference with responsive patterns
- patterns.md - Game design patterns (state machines, ECS, observer)
- services.md - Full service API reference
- animation.md - TweenService, AnimationController, Sequencing
