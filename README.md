--[[
╔══════════════════════════════════════════════════════════════════╗
║              BLADE BALL — NEXUS HUB  v1.0                        ║
║              Made by Minh Scripts                                 ║
║                                                                   ║
║  ✓ 1-Click Accurate Auto Parry (fires ONCE per ball approach)    ║
║  ✓ 100% Accurate Auto Clash Spam (every frame in clash range)    ║
║  ✓ Real Sword Skin Changer (all rarities, 140+ swords)           ║
║  ✓ Loading Screen  ✓ Minh Scripts notification                   ║
║  ✓ X button (confirm + fade)  ✓ Minimize / Restore               ║
║  ✓ Anti-AFK  ✓ Device GUI Fix  ✓ Draggable                      ║
╚══════════════════════════════════════════════════════════════════╝

  PARRY:  GetPropertyChangedSignal fires on every ball position
          change. When the ball crosses the distance threshold
          heading towards you (IsTargeted), ONE click fires.
          parried=true prevents double-clicking. Resets when
          ball moves away (dist > 40) or player dies.

  CLASH:  Ball within ClashDistance + IsTargeted → spam every
          Heartbeat frame. No limit needed — clash window requires
          as many inputs as possible.
]]

-- ════════════════════════════════════════════════════
--  SERVICES
-- ════════════════════════════════════════════════════
local Players             = game:GetService("Players")
local RunService          = game:GetService("RunService")
local UserInputService    = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TweenService        = game:GetService("TweenService")

-- ════════════════════════════════════════════════════
--  CORE
-- ════════════════════════════════════════════════════
local Player = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Balls  = workspace:WaitForChild("Balls", 9e9)

-- ════════════════════════════════════════════════════
--  STATE
-- ════════════════════════════════════════════════════
local State = {
    AutoParry    = true,
    AutoClash    = true,
    SkinChanger  = false,
    SelectedSkin = "Base Sword",
    Parries      = 0,
    FPS          = 60,
    BallSpeedStr = "—",
    Status       = "Waiting...",
}

local ClashDistance  = 14   -- studs
local PARRY_DISTANCE = 10   -- studs — fire parry click when ball is this close

-- ════════════════════════════════════════════════════
--  SWORD DATA
-- ════════════════════════════════════════════════════
local RARITY_COLOR = {
    COMMON    = Color3.fromRGB(100,200,100),
    RARE      = Color3.fromRGB(80, 140,255),
    LEGENDARY = Color3.fromRGB(255,200,50),
    LIMITED   = Color3.fromRGB(220,70, 70),
    UNIQUE    = Color3.fromRGB(180,80, 255),
    LTM       = Color3.fromRGB(80, 220,255),
}
local SwordData = {
    {"Base Sword","COMMON",Color3.fromRGB(160,160,160)},
    {"Dark Blade","COMMON",Color3.fromRGB(40,40,50)},
    {"Pickaxe","COMMON",Color3.fromRGB(120,120,100)},
    {"Massive Lollipop","COMMON",Color3.fromRGB(255,120,180)},
    {"Melon Slice","COMMON",Color3.fromRGB(100,200,80)},
    {"Wooden Hammer","COMMON",Color3.fromRGB(160,100,60)},
    {"Tanto","COMMON",Color3.fromRGB(200,200,200)},
    {"Bone Rapier","COMMON",Color3.fromRGB(240,230,200)},
    {"Stick","COMMON",Color3.fromRGB(140,100,60)},
    {"Baseball Bat","COMMON",Color3.fromRGB(210,170,100)},
    {"Battle Axe","COMMON",Color3.fromRGB(150,150,150)},
    {"Emerald Warden Blade","COMMON",Color3.fromRGB(50,200,100)},
    {"Kitchen Knife","COMMON",Color3.fromRGB(200,200,210)},
    {"Etherium Dagger","RARE",Color3.fromRGB(100,180,255)},
    {"Vengeful Dagger","RARE",Color3.fromRGB(160,60,200)},
    {"Executive Greatsword","RARE",Color3.fromRGB(50,80,160)},
    {"Dragonslayer Blade","RARE",Color3.fromRGB(180,60,40)},
    {"Excalibur","LEGENDARY",Color3.fromRGB(255,230,80)},
    {"Holy Sword","LEGENDARY",Color3.fromRGB(255,255,200)},
    {"Trident","LEGENDARY",Color3.fromRGB(60,160,255)},
    {"Holy Axe","LEGENDARY",Color3.fromRGB(255,220,100)},
    {"Solarus","LEGENDARY",Color3.fromRGB(255,180,0)},
    {"Cyber Blade","LEGENDARY",Color3.fromRGB(0,220,255)},
    {"Aurorum","LEGENDARY",Color3.fromRGB(255,200,120)},
    {"Musara","LEGENDARY",Color3.fromRGB(200,100,255)},
    {"Thunderous Hammer","LEGENDARY",Color3.fromRGB(255,220,0)},
    {"Empyrean Greatblade","LIMITED",Color3.fromRGB(120,60,255)},
    {"Oni Claws","LIMITED",Color3.fromRGB(200,40,40)},
    {"Cyber Scythe","LIMITED",Color3.fromRGB(0,200,255)},
    {"Wavelight Greatblade","LIMITED",Color3.fromRGB(80,200,255)},
    {"Prince Blade","LIMITED",Color3.fromRGB(255,180,60)},
    {"Hallow's Edge","LIMITED",Color3.fromRGB(140,60,200)},
    {"Nebula Scythe","LIMITED",Color3.fromRGB(100,0,180)},
    {"Dual Nebula Scythe","LIMITED",Color3.fromRGB(120,20,200)},
    {"Ether Blade","LIMITED",Color3.fromRGB(80,180,255)},
    {"Dual Ether Blade","LIMITED",Color3.fromRGB(60,160,255)},
    {"Vanguard Shield","LIMITED",Color3.fromRGB(60,120,200)},
    {"Empyrean Fortress","LIMITED",Color3.fromRGB(100,80,220)},
    {"Crescendo","LIMITED",Color3.fromRGB(200,120,255)},
    {"Cyber Thorn","LIMITED",Color3.fromRGB(40,220,120)},
    {"Essence Cleaver","LIMITED",Color3.fromRGB(255,100,200)},
    {"Molten Greatblade","LIMITED",Color3.fromRGB(255,80,20)},
    {"Shattered Sword","LIMITED",Color3.fromRGB(180,180,220)},
    {"Anime Sword","LIMITED",Color3.fromRGB(255,140,200)},
    {"Bowtie Blade","LIMITED",Color3.fromRGB(255,200,80)},
    {"Party Popper","LIMITED",Color3.fromRGB(255,210,100)},
    {"Festive Eclipse Blade","LIMITED",Color3.fromRGB(200,40,100)},
    {"Inferno Emberblade","LIMITED",Color3.fromRGB(255,60,0)},
    {"Gladiator Sword","LIMITED",Color3.fromRGB(200,160,60)},
    {"Siam Ember Axe","LIMITED",Color3.fromRGB(220,80,30)},
    {"Siamese Edgeblade","LIMITED",Color3.fromRGB(200,60,40)},
    {"Gobble Blade","LIMITED",Color3.fromRGB(200,150,60)},
    {"Glacier Shard","LIMITED",Color3.fromRGB(180,230,255)},
    {"Arctic Edge","LIMITED",Color3.fromRGB(200,240,255)},
    {"Dual Frost Saber","LIMITED",Color3.fromRGB(160,230,255)},
    {"Glacial Scythe","LIMITED",Color3.fromRGB(140,220,255)},
    {"Inferno's Edge","LIMITED",Color3.fromRGB(255,70,0)},
    {"Moonlight Bliss","LIMITED",Color3.fromRGB(180,160,255)},
    {"Forsaken Riftide","LIMITED",Color3.fromRGB(80,40,160)},
    {"Light's Edge","LIMITED",Color3.fromRGB(255,255,200)},
    {"Victory Blade","LIMITED",Color3.fromRGB(255,200,60)},
    {"Blade of Exile","LIMITED",Color3.fromRGB(100,60,180)},
    {"The Pendulum","LIMITED",Color3.fromRGB(180,140,255)},
    {"The Covenant","LIMITED",Color3.fromRGB(100,60,220)},
    {"Lament's Blade","LIMITED",Color3.fromRGB(80,40,160)},
    {"Whispering Katana","LIMITED",Color3.fromRGB(220,180,255)},
    {"Evil Slicer","LIMITED",Color3.fromRGB(200,20,60)},
    {"Devil Katana","LIMITED",Color3.fromRGB(220,0,40)},
    {"Steel Fang","LIMITED",Color3.fromRGB(180,180,180)},
    {"Iron Edge","LIMITED",Color3.fromRGB(140,140,140)},
    {"Silver Strike","LIMITED",Color3.fromRGB(200,210,220)},
    {"Dragon's Claw","LIMITED",Color3.fromRGB(200,40,40)},
    {"Eclipse Reaver","LIMITED",Color3.fromRGB(60,20,120)},
    {"Robotic Arm","LIMITED",Color3.fromRGB(140,180,200)},
    {"Cyborg Blade","LIMITED",Color3.fromRGB(80,200,220)},
    {"Dual Cyborg Blade","LIMITED",Color3.fromRGB(60,180,200)},
    {"All-Star Striker","LIMITED",Color3.fromRGB(255,220,80)},
    {"Draconic Bite","LIMITED",Color3.fromRGB(160,20,20)},
    {"Draconic Slasher","LIMITED",Color3.fromRGB(200,50,30)},
    {"Dragon Scythe","LIMITED",Color3.fromRGB(180,30,30)},
    {"Dragon Blade","LIMITED",Color3.fromRGB(200,40,40)},
    {"Ember Blade","LIMITED",Color3.fromRGB(255,100,40)},
    {"Flame Edge","LIMITED",Color3.fromRGB(255,120,0)},
    {"Blazing Saber","LIMITED",Color3.fromRGB(255,140,20)},
    {"Spectral Cleaver","LIMITED",Color3.fromRGB(160,120,255)},
    {"Pumpkin Reaper","LIMITED",Color3.fromRGB(220,100,20)},
    {"Mummy's Curse","LIMITED",Color3.fromRGB(200,180,100)},
    {"Hollow Blade","LIMITED",Color3.fromRGB(50,50,80)},
    {"Nightshade","LIMITED",Color3.fromRGB(60,20,100)},
    {"Nightmare's Slash","LIMITED",Color3.fromRGB(80,20,120)},
    {"Skullsplitter","LIMITED",Color3.fromRGB(200,200,200)},
    {"Headless Horror","LIMITED",Color3.fromRGB(80,20,20)},
    {"Mummy's Slasher","LIMITED",Color3.fromRGB(180,160,80)},
    {"Curse of the Nile","LIMITED",Color3.fromRGB(200,160,60)},
    {"Wicked Reaper","LIMITED",Color3.fromRGB(100,0,80)},
    {"Northern Blade","LIMITED",Color3.fromRGB(180,220,255)},
    {"Glacial Dominance","LIMITED",Color3.fromRGB(160,230,255)},
    {"Polar Fury","LIMITED",Color3.fromRGB(200,240,255)},
    {"Glacial Strike","LIMITED",Color3.fromRGB(140,220,255)},
    {"Frostfire Blade","LIMITED",Color3.fromRGB(100,180,255)},
    {"Harvest Glow","LIMITED",Color3.fromRGB(220,150,60)},
    {"Autumn Sovereign","LIMITED",Color3.fromRGB(200,120,40)},
    {"Noble Fang","LIMITED",Color3.fromRGB(200,180,100)},
    {"Sovereign Edge","LIMITED",Color3.fromRGB(220,200,80)},
    {"Royal Crescent","LIMITED",Color3.fromRGB(255,200,60)},
    {"King's Judgement","LIMITED",Color3.fromRGB(255,210,80)},
    {"Crownslayer","LIMITED",Color3.fromRGB(220,180,60)},
    {"Thronebreaker","LIMITED",Color3.fromRGB(200,160,60)},
    {"Cryo Shard","LIMITED",Color3.fromRGB(160,220,255)},
    {"Candy Cane Cutter","LIMITED",Color3.fromRGB(255,100,100)},
    {"Frostbite Dagger","LIMITED",Color3.fromRGB(160,220,255)},
    {"Crystal Dagger","LIMITED",Color3.fromRGB(200,240,255)},
    {"Aurora Carver","LIMITED",Color3.fromRGB(0,210,180)},
    {"Candycane Cleaver","LIMITED",Color3.fromRGB(255,120,120)},
    {"Holiday Saber","LIMITED",Color3.fromRGB(220,60,60)},
    {"Tinsel Edge","LIMITED",Color3.fromRGB(200,200,100)},
    {"Snowflake Cutter","LIMITED",Color3.fromRGB(200,230,255)},
    {"Pine Shard","LIMITED",Color3.fromRGB(60,160,80)},
    {"Icy Whisper","LIMITED",Color3.fromRGB(180,230,255)},
    {"Frostlight Fang","LIMITED",Color3.fromRGB(160,220,255)},
    {"Blizzard Katana","LIMITED",Color3.fromRGB(180,230,255)},
    {"Emperor Blade","LIMITED",Color3.fromRGB(200,160,60)},
    {"Frosty Dagger","LIMITED",Color3.fromRGB(180,230,255)},
    {"Snowboard Edge","LIMITED",Color3.fromRGB(100,180,255)},
    {"Icy Puck Blade","LIMITED",Color3.fromRGB(160,220,255)},
    {"Blizzard Skater","LIMITED",Color3.fromRGB(180,230,255)},
    {"Crystal Blade","LIMITED",Color3.fromRGB(200,240,255)},
    {"Iced Avenger","LIMITED",Color3.fromRGB(140,200,255)},
    {"Emberflare Blade","LIMITED",Color3.fromRGB(255,100,20)},
    {"Aligned Constellation","LIMITED",Color3.fromRGB(200,200,255)},
    {"Golden Flare","LIMITED",Color3.fromRGB(255,200,60)},
    {"Celebration Edge","LIMITED",Color3.fromRGB(255,180,80)},
    {"Resolution Reaver","LIMITED",Color3.fromRGB(180,140,255)},
    {"Aurora Slash","LIMITED",Color3.fromRGB(0,200,200)},
    {"Arachnoblade","LIMITED",Color3.fromRGB(40,40,60)},
    {"Void Weaver","LIMITED",Color3.fromRGB(80,0,140)},
    {"Sparkler Edge","LIMITED",Color3.fromRGB(255,220,100)},
    {"Countdown Cutter","LIMITED",Color3.fromRGB(180,140,255)},
    {"Celebration Cleaver","LIMITED",Color3.fromRGB(255,200,80)},
    {"Midnight Fang","LIMITED",Color3.fromRGB(60,20,100)},
    {"Firework Edge","LIMITED",Color3.fromRGB(255,120,60)},
    {"Chrono Shard","LIMITED",Color3.fromRGB(140,200,255)},
    {"Eclipse Edge","LIMITED",Color3.fromRGB(60,0,100)},
    {"Lunar Bloom","LIMITED",Color3.fromRGB(200,180,255)},
    {"Eternal Spark","LIMITED",Color3.fromRGB(255,200,120)},
    {"Chrono Edge","LIMITED",Color3.fromRGB(160,210,255)},
    {"Lunar Aegis","LIMITED",Color3.fromRGB(180,160,255)},
    {"Solar Flare Blade","LIMITED",Color3.fromRGB(255,180,0)},
    {"Starlight Spear","LIMITED",Color3.fromRGB(255,255,180)},
    {"Golden Fang","LIMITED",Color3.fromRGB(255,200,60)},
    {"Dual Golden Fang","LIMITED",Color3.fromRGB(255,210,80)},
    {"Void Vanguard","LIMITED",Color3.fromRGB(60,0,120)},
    {"Mecha Axe","LIMITED",Color3.fromRGB(100,200,200)},
    {"Warden's Edge","LIMITED",Color3.fromRGB(80,160,100)},
    {"Voidborn Reaver","LIMITED",Color3.fromRGB(80,0,160)},
    {"Survivor's Fang","LIMITED",Color3.fromRGB(180,160,100)},
    {"Stormpiercer Blade","LIMITED",Color3.fromRGB(120,180,255)},
    {"Gale Sabre","LIMITED",Color3.fromRGB(140,200,255)},
    {"Thunderfall Edge","LIMITED",Color3.fromRGB(220,220,0)},
    {"Obsidian Edge","LIMITED",Color3.fromRGB(30,30,50)},
    {"Skyfang","LIMITED",Color3.fromRGB(100,180,255)},
    {"Ironhowl","LIMITED",Color3.fromRGB(140,140,140)},
    {"Solar Cleaver","LIMITED",Color3.fromRGB(255,180,0)},
    {"Sunset Pastelblade","LIMITED",Color3.fromRGB(255,160,120)},
    {"Titan Fang","LIMITED",Color3.fromRGB(180,140,100)},
    {"Runesteel","LIMITED",Color3.fromRGB(140,160,200)},
    {"Plasma Katana","LIMITED",Color3.fromRGB(80,100,255)},
    {"Coral Greatsword","LIMITED",Color3.fromRGB(255,120,120)},
    {"Avis Scythe","UNIQUE",Color3.fromRGB(180,80,255)},
    {"The Nooblade","UNIQUE",Color3.fromRGB(100,200,100)},
    {"Flowing Katana","UNIQUE",Color3.fromRGB(100,180,255)},
    {"Santa's Wrecker","UNIQUE",Color3.fromRGB(220,60,60)},
    {"Venom Blade","UNIQUE",Color3.fromRGB(60,220,60)},
    {"Ranked Season 1 Top 50","UNIQUE",Color3.fromRGB(255,200,60)},
    {"Ranked Season 9 Top 200","UNIQUE",Color3.fromRGB(180,80,255)},
    {"Cosmic Starblade","LTM",Color3.fromRGB(80,220,255)},
    {"Frostshard Blade","LTM",Color3.fromRGB(160,230,255)},
    {"Dawnblade","LTM",Color3.fromRGB(255,200,100)},
}

-- ════════════════════════════════════════════════════
--  CORE HELPERS
-- ════════════════════════════════════════════════════
local function IsTargeted()
    local c = Player.Character
    return c ~= nil and c:FindFirstChildWhichIsA("Highlight") ~= nil
end

local function GetRootPos()
    local c = Player.Character
    if c then
        local h = c:FindFirstChild("HumanoidRootPart")
        if h then return h.Position end
    end
    return Camera.CFrame.Position
end

-- Single left-click
local function Click()
    VirtualInputManager:SendMouseButtonEvent(0,0,0,true,  game,0)
    VirtualInputManager:SendMouseButtonEvent(0,0,0,false, game,0)
end

local function IsRealBall(o)
    return typeof(o)=="Instance"
        and o:IsA("BasePart")
        and o:IsDescendantOf(Balls)
        and o:GetAttribute("realBall")==true
end

-- ════════════════════════════════════════════════════
--  AUTO PARRY  — ONE click per ball approach
--  Uses GetPropertyChangedSignal("Position") on every
--  real ball. When dist <= PARRY_DISTANCE and targeted
--  → fire ONE click. parried flag prevents re-clicking
--  until the ball moves away (dist > 40).
-- ════════════════════════════════════════════════════
local function TrackBall(ball)
    if not IsRealBall(ball) then return end

    local prevPos  = ball.Position
    local prevTime = tick()
    local parried  = false
    local conns    = {}

    local function Cleanup()
        for _,c in ipairs(conns) do c:Disconnect() end
        conns = {}
    end

    table.insert(conns, ball.Destroying:Connect(Cleanup))

    table.insert(conns, ball:GetPropertyChangedSignal("Position"):Connect(function()
        local now     = tick()
        local ballPos = ball.Position
        local rootPos = GetRootPos()
        local dist    = (ballPos - rootPos).Magnitude

        -- Speed tracking (display only)
        local dt = now - prevTime
        if dt > 0 and dt >= (1/60) then
            local spd = (ballPos - prevPos).Magnitude / dt
            State.BallSpeedStr = math.round(spd).." st/s"
            prevPos  = ballPos
            prevTime = now
        end

        -- Reset parried flag when ball is far (bounced away / new round)
        if dist > 40 then
            parried = false
            return
        end

        -- AUTO PARRY: fire exactly ONE click when in range
        if State.AutoParry and IsTargeted() and not parried and dist <= PARRY_DISTANCE then
            parried = true
            Click()
            State.Parries = State.Parries + 1
            State.Status  = "✓ Parried #"..State.Parries
        end
    end))
end

for _,b in ipairs(Balls:GetChildren()) do task.spawn(TrackBall,b) end
Balls.ChildAdded:Connect(function(b) task.spawn(TrackBall,b) end)

-- ════════════════════════════════════════════════════
--  AUTO CLASH SPAM — every Heartbeat frame in range
-- ════════════════════════════════════════════════════
RunService.Heartbeat:Connect(function()
    if not State.AutoClash then return end
    if not IsTargeted()     then return end
    local root = GetRootPos()
    for _,ball in ipairs(Balls:GetChildren()) do
        if IsRealBall(ball) and (ball.Position-root).Magnitude <= ClashDistance then
            Click()
            State.Status = "⚔ Clashing..."
            return
        end
    end
    if not IsTargeted() then State.Status = "Not targeted" end
end)

-- ════════════════════════════════════════════════════
--  SKIN CHANGER
-- ════════════════════════════════════════════════════
local function GetTint(name)
    for _,e in ipairs(SwordData) do
        if e[1]==name then return e[3] end
    end
    return Color3.fromRGB(160,160,160)
end

local function ApplySkin(name)
    local tint = GetTint(name)
    local function DoBag(bag)
        for _,t in ipairs(bag:GetChildren()) do
            if t:IsA("Tool") then
                t.Name = name
                for _,p in ipairs(t:GetDescendants()) do
                    if p:IsA("BasePart") then p.Color=tint end
                end
            end
        end
    end
    if Player.Character then DoBag(Player.Character) end
    DoBag(Player.Backpack)
    State.Status = "🗡 Skin: "..name
end

Player.CharacterAdded:Connect(function()
    task.wait(1.5)
    if State.SkinChanger then ApplySkin(State.SelectedSkin) end
end)

-- ════════════════════════════════════════════════════
--  ANTI-AFK
-- ════════════════════════════════════════════════════
Player.Idled:Connect(function()
    VirtualInputManager:SendKeyEvent(true,"W",false,game)
    task.wait(0.1)
    VirtualInputManager:SendKeyEvent(false,"W",false,game)
end)

-- ════════════════════════════════════════════════════
--  FPS
-- ════════════════════════════════════════════════════
local dtb={}
RunService.Heartbeat:Connect(function(dt)
    dtb[#dtb+1]=dt
    if #dtb>20 then table.remove(dtb,1) end
    local s=0; for _,v in ipairs(dtb) do s=s+v end
    State.FPS=math.round(#dtb/s)
end)

-- ════════════════════════════════════════════════════
--  GUI
-- ════════════════════════════════════════════════════
do
    -- cleanup old instances
    for _,name in ipairs({"NexusGui","NexusLoadGui","NexusNotifGui"}) do
        local old=Player.PlayerGui:FindFirstChild(name)
        if old then old:Destroy() end
    end

    -- ── Palette ──────────────────────────────────────
    local C = {
        BG      = Color3.fromRGB(8,8,14),
        Panel   = Color3.fromRGB(16,16,26),
        Row     = Color3.fromRGB(24,24,38),
        RowSel  = Color3.fromRGB(32,28,56),
        Accent  = Color3.fromRGB(108,55,245),
        AccentD = Color3.fromRGB(80,35,190),
        Green   = Color3.fromRGB(40,200,100),
        Red     = Color3.fromRGB(220,60,60),
        Yellow  = Color3.fromRGB(255,200,60),
        Text    = Color3.fromRGB(230,230,245),
        Sub     = Color3.fromRGB(120,120,155),
        White   = Color3.fromRGB(255,255,255),
        Black   = Color3.fromRGB(0,0,0),
    }

    -- ════════════════════════════════════════════════
    --  LOADING SCREEN
    -- ════════════════════════════════════════════════
    local LoadGui = Instance.new("ScreenGui")
    LoadGui.Name             = "NexusLoadGui"
    LoadGui.ResetOnSpawn     = false
    LoadGui.IgnoreGuiInset   = true   -- device fix: covers full screen
    LoadGui.ZIndexBehavior   = Enum.ZIndexBehavior.Sibling
    LoadGui.Parent           = Player.PlayerGui

    local LoadBG = Instance.new("Frame")
    LoadBG.Size              = UDim2.new(1,0,1,0)
    LoadBG.BackgroundColor3  = C.BG
    LoadBG.BorderSizePixel   = 0
    LoadBG.ZIndex            = 10
    LoadBG.Parent            = LoadGui

    -- Glowing orb
    local Orb = Instance.new("Frame")
    Orb.Size                 = UDim2.new(0,90,0,90)
    Orb.Position             = UDim2.new(0.5,-45,0.38,-45)
    Orb.BackgroundColor3     = C.Accent
    Orb.BorderSizePixel      = 0
    Orb.ZIndex               = 11
    Orb.Parent               = LoadBG
    Instance.new("UICorner",Orb).CornerRadius = UDim.new(1,0)

    -- Inner orb highlight
    local OrbInner = Instance.new("Frame")
    OrbInner.Size            = UDim2.new(0,50,0,50)
    OrbInner.Position        = UDim2.new(0.5,-25,0.5,-25)
    OrbInner.BackgroundColor3 = Color3.fromRGB(180,140,255)
    OrbInner.BorderSizePixel = 0
    OrbInner.ZIndex          = 12
    OrbInner.Parent          = Orb
    Instance.new("UICorner",OrbInner).CornerRadius = UDim.new(1,0)

    -- Lightning bolt label inside orb
    local OrbIcon = Instance.new("TextLabel")
    OrbIcon.Text             = "⚡"
    OrbIcon.Font             = Enum.Font.GothamBold
    OrbIcon.TextSize         = 32
    OrbIcon.TextColor3       = C.White
    OrbIcon.BackgroundTransparency = 1
    OrbIcon.Size             = UDim2.new(1,0,1,0)
    OrbIcon.ZIndex           = 13
    OrbIcon.Parent           = OrbInner

    -- Title
    local LoadTitle = Instance.new("TextLabel")
    LoadTitle.Text           = "NEXUS HUB"
    LoadTitle.Font           = Enum.Font.GothamBold
    LoadTitle.TextSize       = 28
    LoadTitle.TextColor3     = C.White
    LoadTitle.BackgroundTransparency = 1
    LoadTitle.Size           = UDim2.new(1,0,0,36)
    LoadTitle.Position       = UDim2.new(0,0,0.5,-10)
    LoadTitle.TextXAlignment = Enum.TextXAlignment.Center
    LoadTitle.ZIndex         = 11
    LoadTitle.Parent         = LoadBG

    local LoadSub = Instance.new("TextLabel")
    LoadSub.Text             = "by Minh Scripts"
    LoadSub.Font             = Enum.Font.Gotham
    LoadSub.TextSize         = 13
    LoadSub.TextColor3       = Color3.fromRGB(160,140,220)
    LoadSub.BackgroundTransparency = 1
    LoadSub.Size             = UDim2.new(1,0,0,20)
    LoadSub.Position         = UDim2.new(0,0,0.5,28)
    LoadSub.TextXAlignment   = Enum.TextXAlignment.Center
    LoadSub.ZIndex           = 11
    LoadSub.Parent           = LoadBG

    -- Progress bar track
    local PBarTrack = Instance.new("Frame")
    PBarTrack.Size           = UDim2.new(0,240,0,5)
    PBarTrack.Position       = UDim2.new(0.5,-120,0.5,60)
    PBarTrack.BackgroundColor3 = Color3.fromRGB(30,30,50)
    PBarTrack.BorderSizePixel = 0
    PBarTrack.ZIndex         = 11
    PBarTrack.Parent         = LoadBG
    Instance.new("UICorner",PBarTrack).CornerRadius = UDim.new(1,0)

    local PBarFill = Instance.new("Frame")
    PBarFill.Size            = UDim2.new(0,0,1,0)
    PBarFill.BackgroundColor3 = C.Accent
    PBarFill.BorderSizePixel = 0
    PBarFill.ZIndex          = 12
    PBarFill.Parent          = PBarTrack
    Instance.new("UICorner",PBarFill).CornerRadius = UDim.new(1,0)

    local LoadStatus = Instance.new("TextLabel")
    LoadStatus.Text          = "Initializing..."
    LoadStatus.Font          = Enum.Font.Gotham
    LoadStatus.TextSize      = 11
    LoadStatus.TextColor3    = C.Sub
    LoadStatus.BackgroundTransparency = 1
    LoadStatus.Size          = UDim2.new(1,0,0,18)
    LoadStatus.Position      = UDim2.new(0,0,0.5,72)
    LoadStatus.TextXAlignment = Enum.TextXAlignment.Center
    LoadStatus.ZIndex        = 11
    LoadStatus.Parent        = LoadBG

    -- Animate orb pulse
    local function PulseOrb()
        TweenService:Create(Orb, TweenInfo.new(0.8,Enum.EasingStyle.Sine,Enum.EasingDirection.InOut,-1,true),
            {BackgroundColor3=Color3.fromRGB(138,90,255)}):Play()
    end
    PulseOrb()

    -- Loading steps
    local steps = {
        {0.15, "Loading sword data..."},
        {0.35, "Hooking ball events..."},
        {0.55, "Setting up parry..."},
        {0.75, "Building interface..."},
        {0.95, "Almost ready..."},
        {1.00, "Done!"},
    }

    -- ════════════════════════════════════════════════
    --  MAIN GUI  (built while loading screen shows)
    -- ════════════════════════════════════════════════
    local W, H = 310, 420
    local TITLEH = 40
    local TABH   = 34

    local SG = Instance.new("ScreenGui")
    SG.Name             = "NexusGui"
    SG.ResetOnSpawn     = false
    SG.IgnoreGuiInset   = true   -- ← device fix: no notch/inset offset
    SG.ZIndexBehavior   = Enum.ZIndexBehavior.Sibling
    SG.Enabled          = false  -- hidden until loading done
    SG.Parent           = Player.PlayerGui

    -- Shadow
    local Shad = Instance.new("Frame")
    Shad.Size               = UDim2.new(0,W+8,0,H+8)
    Shad.Position           = UDim2.new(0,16,0,16)
    Shad.BackgroundColor3   = C.Black
    Shad.BackgroundTransparency = 0.5
    Shad.BorderSizePixel    = 0
    Shad.Parent             = SG
    Instance.new("UICorner",Shad).CornerRadius = UDim.new(0,14)

    -- Main window
    local Win = Instance.new("Frame")
    Win.Size                = UDim2.new(0,W,0,H)
    Win.Position            = UDim2.new(0,20,0,20)
    Win.BackgroundColor3    = C.BG
    Win.BorderSizePixel     = 0
    Win.ClipsDescendants    = true
    Win.Parent              = SG
    Instance.new("UICorner",Win).CornerRadius = UDim.new(0,12)

    -- ── Title bar ──────────────────────────────────
    local TBar = Instance.new("Frame")
    TBar.Name               = "TBar"
    TBar.Size               = UDim2.new(1,0,0,TITLEH)
    TBar.BackgroundColor3   = C.Accent
    TBar.BorderSizePixel    = 0
    TBar.Parent             = Win
    Instance.new("UICorner",TBar).CornerRadius = UDim.new(0,12)
    -- fill lower corners of title bar
    local TBarFill = Instance.new("Frame")
    TBarFill.Size           = UDim2.new(1,0,0,14)
    TBarFill.Position       = UDim2.new(0,0,1,-14)
    TBarFill.BackgroundColor3 = C.Accent
    TBarFill.BorderSizePixel = 0
    TBarFill.Parent         = TBar

    local TTL = Instance.new("TextLabel")
    TTL.Text                = "⚡  NEXUS HUB"
    TTL.Font                = Enum.Font.GothamBold
    TTL.TextSize            = 15
    TTL.TextColor3          = C.White
    TTL.BackgroundTransparency = 1
    TTL.Size                = UDim2.new(1,-100,1,0)
    TTL.Position            = UDim2.new(0,12,0,0)
    TTL.TextXAlignment      = Enum.TextXAlignment.Left
    TTL.Parent              = TBar

    local TVer = Instance.new("TextLabel")
    TVer.Text               = "v1.0"
    TVer.Font               = Enum.Font.Gotham
    TVer.TextSize           = 10
    TVer.TextColor3         = Color3.fromRGB(200,180,255)
    TVer.BackgroundTransparency = 1
    TVer.Size               = UDim2.new(0,30,1,0)
    TVer.Position           = UDim2.new(1,-118,0,0)
    TVer.TextXAlignment     = Enum.TextXAlignment.Center
    TVer.Parent             = TBar

    -- ── Minimize button  "—"  ──────────────────────
    local MinBtn = Instance.new("TextButton")
    MinBtn.Size             = UDim2.new(0,26,0,22)
    MinBtn.Position         = UDim2.new(1,-60,0.5,-11)
    MinBtn.BackgroundColor3 = Color3.fromRGB(50,30,110)
    MinBtn.BorderSizePixel  = 0
    MinBtn.Font             = Enum.Font.GothamBold
    MinBtn.TextSize         = 16
    MinBtn.TextColor3       = C.White
    MinBtn.Text             = "—"
    MinBtn.Parent           = TBar
    Instance.new("UICorner",MinBtn).CornerRadius = UDim.new(0,6)

    -- ── Close button  "✕"  ────────────────────────
    local CloseBtn = Instance.new("TextButton")
    CloseBtn.Size           = UDim2.new(0,26,0,22)
    CloseBtn.Position       = UDim2.new(1,-30,0.5,-11)
    CloseBtn.BackgroundColor3 = Color3.fromRGB(180,40,40)
    CloseBtn.BorderSizePixel = 0
    CloseBtn.Font           = Enum.Font.GothamBold
    CloseBtn.TextSize       = 14
    CloseBtn.TextColor3     = C.White
    CloseBtn.Text           = "✕"
    CloseBtn.Parent         = TBar
    Instance.new("UICorner",CloseBtn).CornerRadius = UDim.new(0,6)

    -- ── Body (everything below title bar) ──────────
    local Body = Instance.new("Frame")
    Body.Size               = UDim2.new(1,0,1,-TITLEH)
    Body.Position           = UDim2.new(0,0,0,TITLEH)
    Body.BackgroundTransparency = 1
    Body.BorderSizePixel    = 0
    Body.Parent             = Win

    -- ── Dragging ───────────────────────────────────
    do
        local drag,ds,sp
        TBar.InputBegan:Connect(function(i)
            if i.UserInputType==Enum.UserInputType.MouseButton1 or
               i.UserInputType==Enum.UserInputType.Touch then
                drag=true; ds=i.Position; sp=Win.Position
            end
        end)
        TBar.InputEnded:Connect(function(i)
            if i.UserInputType==Enum.UserInputType.MouseButton1 or
               i.UserInputType==Enum.UserInputType.Touch then drag=false end
        end)
        UserInputService.InputChanged:Connect(function(i)
            if not drag then return end
            if i.UserInputType==Enum.UserInputType.MouseMovement or
               i.UserInputType==Enum.UserInputType.Touch then
                local d=i.Position-ds
                local nx=math.clamp(sp.X.Offset+d.X, 0, workspace.CurrentCamera.ViewportSize.X-W-8)
                local ny=math.clamp(sp.Y.Offset+d.Y, 0, workspace.CurrentCamera.ViewportSize.Y-TITLEH-8)
                Win.Position =UDim2.new(0,nx,0,ny)
                Shad.Position=UDim2.new(0,nx-4,0,ny-4)
            end
        end)
    end

    -- ── Minimize logic ─────────────────────────────
    local minimized = false

    local function SetMinimized(state)
        minimized = state
        if minimized then
            -- shrink to title bar only
            MinBtn.Text = "+"
            TweenService:Create(Win, TweenInfo.new(0.18,Enum.EasingStyle.Quad), {
                Size = UDim2.new(0,W,0,TITLEH)
            }):Play()
            Body.Visible = false
            Shad.Size = UDim2.new(0,W+8,0,TITLEH+8)
        else
            -- restore
            MinBtn.Text = "—"
            Body.Visible = true
            TweenService:Create(Win, TweenInfo.new(0.18,Enum.EasingStyle.Quad), {
                Size = UDim2.new(0,W,0,H)
            }):Play()
            Shad.Size = UDim2.new(0,W+8,0,H+8)
        end
    end

    MinBtn.MouseButton1Click:Connect(function()
        SetMinimized(not minimized)
    end)

    -- ── Confirmation dialog (for X button) ─────────
    -- Overlay that darkens the whole screen
    local ConfirmOverlay = Instance.new("Frame")
    ConfirmOverlay.Size             = UDim2.new(1,0,1,0)
    ConfirmOverlay.BackgroundColor3 = C.Black
    ConfirmOverlay.BackgroundTransparency = 1   -- starts invisible
    ConfirmOverlay.BorderSizePixel  = 0
    ConfirmOverlay.Visible          = false
    ConfirmOverlay.ZIndex           = 20
    ConfirmOverlay.Parent           = SG

    local ConfirmBox = Instance.new("Frame")
    ConfirmBox.Size             = UDim2.new(0,260,0,120)
    ConfirmBox.Position         = UDim2.new(0.5,-130,0.5,-60)
    ConfirmBox.BackgroundColor3 = Color3.fromRGB(16,16,28)
    ConfirmBox.BorderSizePixel  = 0
    ConfirmBox.ZIndex           = 21
    ConfirmBox.Parent           = ConfirmOverlay
    Instance.new("UICorner",ConfirmBox).CornerRadius = UDim.new(0,12)

    local ConfirmTitle = Instance.new("TextLabel")
    ConfirmTitle.Text           = "Close Nexus Hub?"
    ConfirmTitle.Font           = Enum.Font.GothamBold
    ConfirmTitle.TextSize       = 15
    ConfirmTitle.TextColor3     = C.White
    ConfirmTitle.BackgroundTransparency = 1
    ConfirmTitle.Size           = UDim2.new(1,0,0,36)
    ConfirmTitle.Position       = UDim2.new(0,0,0,8)
    ConfirmTitle.TextXAlignment = Enum.TextXAlignment.Center
    ConfirmTitle.ZIndex         = 22
    ConfirmTitle.Parent         = ConfirmBox

    local ConfirmSub = Instance.new("TextLabel")
    ConfirmSub.Text             = "This will destroy the GUI."
    ConfirmSub.Font             = Enum.Font.Gotham
    ConfirmSub.TextSize         = 12
    ConfirmSub.TextColor3       = C.Sub
    ConfirmSub.BackgroundTransparency = 1
    ConfirmSub.Size             = UDim2.new(1,0,0,20)
    ConfirmSub.Position         = UDim2.new(0,0,0,38)
    ConfirmSub.TextXAlignment   = Enum.TextXAlignment.Center
    ConfirmSub.ZIndex           = 22
    ConfirmSub.Parent           = ConfirmBox

    -- YES button
    local YesBtn = Instance.new("TextButton")
    YesBtn.Size             = UDim2.new(0,100,0,32)
    YesBtn.Position         = UDim2.new(0.5,-108,1,-44)
    YesBtn.BackgroundColor3 = Color3.fromRGB(180,40,40)
    YesBtn.BorderSizePixel  = 0
    YesBtn.Font             = Enum.Font.GothamBold
    YesBtn.TextSize         = 13
    YesBtn.TextColor3       = C.White
    YesBtn.Text             = "Yes, close"
    YesBtn.ZIndex           = 22
    YesBtn.Parent           = ConfirmBox
    Instance.new("UICorner",YesBtn).CornerRadius = UDim.new(0,8)

    -- NO button
    local NoBtn = Instance.new("TextButton")
    NoBtn.Size              = UDim2.new(0,100,0,32)
    NoBtn.Position          = UDim2.new(0.5,8,1,-44)
    NoBtn.BackgroundColor3  = Color3.fromRGB(40,40,60)
    NoBtn.BorderSizePixel   = 0
    NoBtn.Font              = Enum.Font.GothamBold
    NoBtn.TextSize          = 13
    NoBtn.TextColor3        = C.Sub
    NoBtn.Text              = "Cancel"
    NoBtn.ZIndex            = 22
    NoBtn.Parent            = ConfirmBox
    Instance.new("UICorner",NoBtn).CornerRadius = UDim.new(0,8)

    local function ShowConfirm(visible)
        ConfirmOverlay.Visible = visible
        TweenService:Create(ConfirmOverlay, TweenInfo.new(0.2), {
            BackgroundTransparency = visible and 0.45 or 1
        }):Play()
        if visible then
            ConfirmBox.Position = UDim2.new(0.5,-130,0.4,-60)
            TweenService:Create(ConfirmBox, TweenInfo.new(0.2,Enum.EasingStyle.Back,Enum.EasingDirection.Out), {
                Position = UDim2.new(0.5,-130,0.5,-60)
            }):Play()
        end
    end

    CloseBtn.MouseButton1Click:Connect(function()
        ShowConfirm(true)
    end)
    NoBtn.MouseButton1Click:Connect(function()
        ShowConfirm(false)
    end)
    YesBtn.MouseButton1Click:Connect(function()
        -- fade out then destroy
        TweenService:Create(SG, TweenInfo.new(0.3), {}):Play()
        TweenService:Create(Win, TweenInfo.new(0.3), {
            BackgroundTransparency = 1,
            Size = UDim2.new(0,W,0,0)
        }):Play()
        TweenService:Create(Shad, TweenInfo.new(0.3), {BackgroundTransparency=1}):Play()
        task.delay(0.35, function()
            SG:Destroy()
        end)
    end)

    -- ── Tabs ───────────────────────────────────────
    local TabBar = Instance.new("Frame")
    TabBar.Size             = UDim2.new(1,0,0,TABH)
    TabBar.BackgroundColor3 = C.Panel
    TabBar.BorderSizePixel  = 0
    TabBar.Parent           = Body
    Instance.new("UIListLayout",TabBar).FillDirection = Enum.FillDirection.Horizontal

    local Cont = Instance.new("Frame")
    Cont.Size               = UDim2.new(1,0,1,-(TABH+30))
    Cont.Position           = UDim2.new(0,0,0,TABH)
    Cont.BackgroundTransparency = 1
    Cont.BorderSizePixel    = 0
    Cont.ClipsDescendants   = true
    Cont.Parent             = Body

    local Pages = {}
    local function MakePage(name)
        local pg = Instance.new("ScrollingFrame")
        pg.Size             = UDim2.new(1,0,1,0)
        pg.BackgroundTransparency = 1
        pg.BorderSizePixel  = 0
        pg.ScrollBarThickness = 3
        pg.ScrollBarImageColor3 = C.Accent
        pg.Visible          = false
        pg.CanvasSize       = UDim2.new(0,0,0,0)
        pg.AutomaticCanvasSize = Enum.AutomaticSize.Y
        pg.Parent           = Cont
        local ll = Instance.new("UIListLayout")
        ll.SortOrder        = Enum.SortOrder.LayoutOrder
        ll.Padding          = UDim.new(0,5)
        ll.Parent           = pg
        local pad = Instance.new("UIPadding")
        pad.PaddingLeft=UDim.new(0,10); pad.PaddingRight=UDim.new(0,10)
        pad.PaddingTop=UDim.new(0,8); pad.Parent=pg
        Pages[name] = pg
        return pg
    end

    local PMain  = MakePage("Main")
    local PClash = MakePage("Clash")
    local PSkin  = MakePage("Skins")

    local activeTab = {nil,nil}
    local function MakeTab(label,pageName,order)
        local btn = Instance.new("TextButton")
        btn.Size            = UDim2.new(0,W/3,1,0)
        btn.BackgroundColor3 = C.Panel
        btn.BorderSizePixel = 0
        btn.Font            = Enum.Font.GothamBold
        btn.TextSize        = 12
        btn.TextColor3      = C.Sub
        btn.Text            = label
        btn.LayoutOrder     = order
        btn.Parent          = TabBar
        local bar = Instance.new("Frame")
        bar.Size            = UDim2.new(0.65,0,0,2)
        bar.Position        = UDim2.new(0.175,0,1,-2)
        bar.BackgroundColor3 = C.Accent
        bar.BorderSizePixel = 0
        bar.Visible         = false
        bar.Parent          = btn
        Instance.new("UICorner",bar).CornerRadius = UDim.new(1,0)
        local function Activate()
            if activeTab[1] then activeTab[1].TextColor3=C.Sub; activeTab[2].Visible=false end
            btn.TextColor3=C.White; bar.Visible=true; activeTab={btn,bar}
            for n,pg in pairs(Pages) do pg.Visible=(n==pageName) end
        end
        btn.MouseButton1Click:Connect(Activate)
        if order==1 then Activate() end
    end
    MakeTab("PARRY","Main",1); MakeTab("CLASH","Clash",2); MakeTab("SKINS","Skins",3)

    -- ── Widget helpers ─────────────────────────────
    local function MakeSection(page,text,order)
        local l = Instance.new("TextLabel")
        l.Text=text; l.Font=Enum.Font.GothamBold; l.TextSize=10
        l.TextColor3=C.Sub; l.BackgroundTransparency=1
        l.Size=UDim2.new(1,0,0,16); l.TextXAlignment=Enum.TextXAlignment.Left
        l.LayoutOrder=order; l.Parent=page
    end

    local function MakeToggle(page,label,key,order,cb)
        local row=Instance.new("Frame")
        row.Size=UDim2.new(1,0,0,42); row.BackgroundColor3=C.Row
        row.BorderSizePixel=0; row.LayoutOrder=order; row.Parent=page
        Instance.new("UICorner",row).CornerRadius=UDim.new(0,8)
        local lbl=Instance.new("TextLabel")
        lbl.Text=label; lbl.Font=Enum.Font.GothamBold; lbl.TextSize=13
        lbl.TextColor3=C.Text; lbl.BackgroundTransparency=1
        lbl.Size=UDim2.new(1,-62,1,0); lbl.Position=UDim2.new(0,12,0,0)
        lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.Parent=row
        local pill=Instance.new("Frame")
        pill.Size=UDim2.new(0,42,0,22); pill.Position=UDim2.new(1,-52,0.5,-11)
        pill.BackgroundColor3=State[key] and C.Green or C.Red
        pill.BorderSizePixel=0; pill.Parent=row
        Instance.new("UICorner",pill).CornerRadius=UDim.new(1,0)
        local knob=Instance.new("Frame")
        knob.Size=UDim2.new(0,16,0,16)
        knob.Position=State[key] and UDim2.new(1,-20,0.5,-8) or UDim2.new(0,3,0.5,-8)
        knob.BackgroundColor3=C.White; knob.BorderSizePixel=0; knob.Parent=pill
        Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
        local function Refresh()
            local on=State[key]
            TweenService:Create(pill,TweenInfo.new(0.14),{BackgroundColor3=on and C.Green or C.Red}):Play()
            TweenService:Create(knob,TweenInfo.new(0.14),{
                Position=on and UDim2.new(1,-20,0.5,-8) or UDim2.new(0,3,0.5,-8)
            }):Play()
        end
        local hb=Instance.new("TextButton")
        hb.Size=UDim2.new(1,0,1,0); hb.BackgroundTransparency=1; hb.Text=""; hb.Parent=row
        hb.MouseButton1Click:Connect(function()
            State[key]=not State[key]; Refresh(); if cb then cb(State[key]) end
        end)
    end

    local function MakeStat(page,label,order)
        local row=Instance.new("Frame")
        row.Size=UDim2.new(1,0,0,34); row.BackgroundColor3=C.Row
        row.BorderSizePixel=0; row.LayoutOrder=order; row.Parent=page
        Instance.new("UICorner",row).CornerRadius=UDim.new(0,8)
        local l=Instance.new("TextLabel")
        l.Text=label; l.Font=Enum.Font.Gotham; l.TextSize=12; l.TextColor3=C.Sub
        l.BackgroundTransparency=1; l.Size=UDim2.new(0.55,0,1,0); l.Position=UDim2.new(0,12,0,0)
        l.TextXAlignment=Enum.TextXAlignment.Left; l.Parent=row
        local v=Instance.new("TextLabel")
        v.Text="—"; v.Font=Enum.Font.GothamBold; v.TextSize=12; v.TextColor3=C.White
        v.BackgroundTransparency=1; v.Size=UDim2.new(0.44,0,1,0); v.Position=UDim2.new(0.55,0,0,0)
        v.TextXAlignment=Enum.TextXAlignment.Right; v.Parent=row
        return v
    end

    local function MakeSlider(page,label,minV,maxV,def,order,cb)
        local row=Instance.new("Frame")
        row.Size=UDim2.new(1,0,0,58); row.BackgroundColor3=C.Row
        row.BorderSizePixel=0; row.LayoutOrder=order; row.Parent=page
        Instance.new("UICorner",row).CornerRadius=UDim.new(0,8)
        local lbl=Instance.new("TextLabel")
        lbl.Text=label; lbl.Font=Enum.Font.GothamBold; lbl.TextSize=12; lbl.TextColor3=C.Text
        lbl.BackgroundTransparency=1; lbl.Size=UDim2.new(0.65,0,0,26); lbl.Position=UDim2.new(0,12,0,4)
        lbl.TextXAlignment=Enum.TextXAlignment.Left; lbl.Parent=row
        local vl=Instance.new("TextLabel")
        vl.Text=tostring(def); vl.Font=Enum.Font.GothamBold; vl.TextSize=12; vl.TextColor3=C.Accent
        vl.BackgroundTransparency=1; vl.Size=UDim2.new(0.3,0,0,26); vl.Position=UDim2.new(0.68,0,0,4)
        vl.TextXAlignment=Enum.TextXAlignment.Right; vl.Parent=row
        local track=Instance.new("Frame")
        track.Size=UDim2.new(1,-24,0,6); track.Position=UDim2.new(0,12,0,40)
        track.BackgroundColor3=C.BG; track.BorderSizePixel=0; track.Parent=row
        Instance.new("UICorner",track).CornerRadius=UDim.new(1,0)
        local p0=(def-minV)/(maxV-minV)
        local fill=Instance.new("Frame")
        fill.Size=UDim2.new(p0,0,1,0); fill.BackgroundColor3=C.Accent
        fill.BorderSizePixel=0; fill.Parent=track
        Instance.new("UICorner",fill).CornerRadius=UDim.new(1,0)
        local knob=Instance.new("TextButton")
        knob.Size=UDim2.new(0,14,0,14); knob.Position=UDim2.new(p0,-7,0.5,-7)
        knob.BackgroundColor3=C.White; knob.BorderSizePixel=0; knob.Text=""; knob.Parent=track
        Instance.new("UICorner",knob).CornerRadius=UDim.new(1,0)
        local sliding=false
        knob.InputBegan:Connect(function(i)
            if i.UserInputType==Enum.UserInputType.MouseButton1 or
               i.UserInputType==Enum.UserInputType.Touch then sliding=true end
        end)
        UserInputService.InputEnded:Connect(function(i)
            if i.UserInputType==Enum.UserInputType.MouseButton1 or
               i.UserInputType==Enum.UserInputType.Touch then sliding=false end
        end)
        UserInputService.InputChanged:Connect(function(i)
            if not sliding then return end
            if i.UserInputType==Enum.UserInputType.MouseMovement or
               i.UserInputType==Enum.UserInputType.Touch then
                local p=math.clamp((i.Position.X-track.AbsolutePosition.X)/track.AbsoluteSize.X,0,1)
                local nv=math.round(minV+(maxV-minV)*p)
                fill.Size=UDim2.new(p,0,1,0); knob.Position=UDim2.new(p,-7,0.5,-7)
                vl.Text=tostring(nv); if cb then cb(nv) end
            end
        end)
    end

    -- ═══════════════════════
    --  MAIN PAGE
    -- ═══════════════════════
    MakeSection(PMain,"AUTO PARRY",1)
    MakeToggle(PMain,"Auto Parry","AutoParry",2)

    local pdRow=Instance.new("Frame")
    pdRow.Size=UDim2.new(1,0,0,56); pdRow.BackgroundColor3=C.Row
    pdRow.BorderSizePixel=0; pdRow.LayoutOrder=3; pdRow.Parent=PMain
    Instance.new("UICorner",pdRow).CornerRadius=UDim.new(0,8)
    local pdT=Instance.new("TextLabel")
    pdT.Text="Parry Distance"; pdT.Font=Enum.Font.GothamBold; pdT.TextSize=12
    pdT.TextColor3=C.Text; pdT.BackgroundTransparency=1
    pdT.Size=UDim2.new(0.65,0,0,26); pdT.Position=UDim2.new(0,12,0,4)
    pdT.TextXAlignment=Enum.TextXAlignment.Left; pdT.Parent=pdRow
    local pdV=Instance.new("TextLabel")
    pdV.Text=tostring(PARRY_DISTANCE).." st"; pdV.Font=Enum.Font.GothamBold; pdV.TextSize=12
    pdV.TextColor3=C.Accent; pdV.BackgroundTransparency=1
    pdV.Size=UDim2.new(0.3,0,0,26); pdV.Position=UDim2.new(0.68,0,0,4)
    pdV.TextXAlignment=Enum.TextXAlignment.Right; pdV.Parent=pdRow
    local pdTrack=Instance.new("Frame")
    pdTrack.Size=UDim2.new(1,-24,0,6); pdTrack.Position=UDim2.new(0,12,0,38)
    pdTrack.BackgroundColor3=C.BG; pdTrack.BorderSizePixel=0; pdTrack.Parent=pdRow
    Instance.new("UICorner",pdTrack).CornerRadius=UDim.new(1,0)
    local pdFill=Instance.new("Frame")
    pdFill.Size=UDim2.new((PARRY_DISTANCE-4)/(30-4),0,1,0)
    pdFill.BackgroundColor3=C.Accent; pdFill.BorderSizePixel=0; pdFill.Parent=pdTrack
    Instance.new("UICorner",pdFill).CornerRadius=UDim.new(1,0)
    local pdKnob=Instance.new("TextButton")
    pdKnob.Size=UDim2.new(0,14,0,14)
    pdKnob.Position=UDim2.new((PARRY_DISTANCE-4)/(30-4),-7,0.5,-7)
    pdKnob.BackgroundColor3=C.White; pdKnob.BorderSizePixel=0; pdKnob.Text=""; pdKnob.Parent=pdTrack
    Instance.new("UICorner",pdKnob).CornerRadius=UDim.new(1,0)
    local pdSliding=false
    pdKnob.InputBegan:Connect(function(i)
        if i.UserInputType==Enum.UserInputType.MouseButton1 or
           i.UserInputType==Enum.UserInputType.Touch then pdSliding=true end
    end)
    UserInputService.InputEnded:Connect(function(i)
        if i.UserInputType==Enum.UserInputType.MouseButton1 or
           i.UserInputType==Enum.UserInputType.Touch then pdSliding=false end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if not pdSliding then return end
        if i.UserInputType==Enum.UserInputType.MouseMovement or
           i.UserInputType==Enum.UserInputType.Touch then
            local p=math.clamp((i.Position.X-pdTrack.AbsolutePosition.X)/pdTrack.AbsoluteSize.X,0,1)
            local nv=math.round(4+(30-4)*p)
            PARRY_DISTANCE=nv
            pdFill.Size=UDim2.new(p,0,1,0); pdKnob.Position=UDim2.new(p,-7,0.5,-7)
            pdV.Text=tostring(nv).." st"
        end
    end)

    MakeSection(PMain,"LIVE STATS",4)
    local sParr  = MakeStat(PMain,"Total Parries",5)
    local sFPS   = MakeStat(PMain,"FPS",6)
    local sSpeed = MakeStat(PMain,"Ball Speed",7)
    local sStat  = MakeStat(PMain,"Status",8)

    -- ═══════════════════════
    --  CLASH PAGE
    -- ═══════════════════════
    MakeSection(PClash,"AUTO CLASH SPAM",1)
    MakeToggle(PClash,"Auto Clash Spam","AutoClash",2)
    MakeSection(PClash,"CLASH DETECT RANGE",4)
    MakeSlider(PClash,"Distance (studs)",4,30,14,5,function(v) ClashDistance=v end)
    local cn=Instance.new("TextLabel")
    cn.Text="Spams click every frame when ball\nenters clash range while targeted."
    cn.Font=Enum.Font.Gotham; cn.TextSize=11; cn.TextColor3=C.Sub
    cn.BackgroundTransparency=1; cn.Size=UDim2.new(1,0,0,36)
    cn.TextWrapped=true; cn.TextXAlignment=Enum.TextXAlignment.Left
    cn.LayoutOrder=6; cn.Parent=PClash

    -- ═══════════════════════
    --  SKINS PAGE
    -- ═══════════════════════
    MakeSection(PSkin,"SWORD SKIN CHANGER",1)
    MakeToggle(PSkin,"Enable Skin Changer","SkinChanger",2,function(on)
        if on then ApplySkin(State.SelectedSkin) end
    end)
    local sn=Instance.new("TextLabel")
    sn.Text="⚠ Renames sword client-side.\n   Must own skin server-side for others to see."
    sn.Font=Enum.Font.Gotham; sn.TextSize=10; sn.TextColor3=C.Yellow
    sn.BackgroundTransparency=1; sn.Size=UDim2.new(1,0,0,36)
    sn.TextWrapped=true; sn.TextXAlignment=Enum.TextXAlignment.Left
    sn.LayoutOrder=3; sn.Parent=PSkin
    MakeSection(PSkin,"SELECT SWORD  ("..#SwordData..")",4)
    local activeSkinBtn=nil
    local rarityOrder={"COMMON","RARE","LEGENDARY","LIMITED","UNIQUE","LTM"}
    local rarityLabel={COMMON="🟩 COMMON",RARE="🔵 RARE",LEGENDARY="🟡 LEGENDARY",
        LIMITED="🔴 LIMITED",UNIQUE="💎 UNIQUE",LTM="🏆 LTM LEADERBOARD"}
    local lo=5
    for _,rarity in ipairs(rarityOrder) do
        local rh=Instance.new("TextLabel")
        rh.Text=rarityLabel[rarity]; rh.Font=Enum.Font.GothamBold; rh.TextSize=11
        rh.TextColor3=RARITY_COLOR[rarity]; rh.BackgroundTransparency=1
        rh.Size=UDim2.new(1,0,0,18); rh.TextXAlignment=Enum.TextXAlignment.Left
        rh.LayoutOrder=lo; rh.Parent=PSkin; lo=lo+1
        for _,e in ipairs(SwordData) do
            local nm,er,tint=e[1],e[2],e[3]
            if er==rarity then
                local isSel=(nm==State.SelectedSkin)
                local btn=Instance.new("TextButton")
                btn.Size=UDim2.new(1,0,0,32); btn.BorderSizePixel=0; btn.LayoutOrder=lo
                btn.Font=Enum.Font.GothamBold; btn.TextSize=12
                btn.TextColor3=isSel and C.White or C.Text
                btn.BackgroundColor3=isSel and C.RowSel or C.Row
                btn.Text="    "..nm; btn.TextXAlignment=Enum.TextXAlignment.Left; btn.Parent=PSkin
                Instance.new("UICorner",btn).CornerRadius=UDim.new(0,7)
                local strip=Instance.new("Frame")
                strip.Size=UDim2.new(0,3,1,0); strip.BackgroundColor3=RARITY_COLOR[rarity]
                strip.BorderSizePixel=0; strip.Parent=btn
                Instance.new("UICorner",strip).CornerRadius=UDim.new(0,3)
                local chip=Instance.new("Frame")
                chip.Size=UDim2.new(0,12,0,12); chip.Position=UDim2.new(1,-22,0.5,-6)
                chip.BackgroundColor3=tint; chip.BorderSizePixel=0; chip.Parent=btn
                Instance.new("UICorner",chip).CornerRadius=UDim.new(1,0)
                if isSel then activeSkinBtn=btn end
                btn.MouseButton1Click:Connect(function()
                    if activeSkinBtn then
                        activeSkinBtn.TextColor3=C.Text
                        activeSkinBtn.BackgroundColor3=C.Row
                    end
                    btn.TextColor3=C.White; btn.BackgroundColor3=C.RowSel
                    activeSkinBtn=btn; State.SelectedSkin=nm
                    if State.SkinChanger then ApplySkin(nm) end
                end)
                lo=lo+1
            end
        end
    end

    -- ── Status bar ─────────────────────────────────
    local SB=Instance.new("Frame")
    SB.Size=UDim2.new(1,0,0,30); SB.Position=UDim2.new(0,0,1,-30)
    SB.BackgroundColor3=C.Panel; SB.BorderSizePixel=0; SB.Parent=Body
    local SBL=Instance.new("TextLabel")
    SBL.Font=Enum.Font.Gotham; SBL.TextSize=11; SBL.TextColor3=C.Sub
    SBL.BackgroundTransparency=1; SBL.Size=UDim2.new(1,-12,1,0)
    SBL.Position=UDim2.new(0,12,0,0); SBL.TextXAlignment=Enum.TextXAlignment.Left; SBL.Parent=SB

    -- live stat updates
    RunService.Heartbeat:Connect(function()
        sParr.Text  = tostring(State.Parries)
        sFPS.Text   = tostring(State.FPS).." fps"
        sSpeed.Text = State.BallSpeedStr
        sStat.Text  = State.Status
        SBL.TextColor3 = State.Status:find("Parried") and C.Green
            or State.Status:find("Clash") and C.Yellow or C.Sub
        SBL.Text = "⚡ "..State.Status
    end)

    -- ════════════════════════════════════════════════
    --  NOTIFICATION — "Minh Scripts"
    --  bottom-right, slides in after GUI opens
    -- ════════════════════════════════════════════════
    local NotifGui = Instance.new("ScreenGui")
    NotifGui.Name           = "NexusNotifGui"
    NotifGui.ResetOnSpawn   = false
    NotifGui.IgnoreGuiInset = true   -- device fix
    NotifGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    NotifGui.Parent         = Player.PlayerGui

    local Notif = Instance.new("Frame")
    Notif.Size              = UDim2.new(0,240,0,60)
    -- start off-screen to the right, then slide in
    Notif.Position          = UDim2.new(1,20,1,-80)
    Notif.BackgroundColor3  = Color3.fromRGB(14,14,22)
    Notif.BorderSizePixel   = 0
    Notif.ZIndex            = 5
    Notif.Parent            = NotifGui
    Instance.new("UICorner",Notif).CornerRadius = UDim.new(0,10)

    -- Left accent stripe
    local NStripe = Instance.new("Frame")
    NStripe.Size            = UDim2.new(0,4,1,0)
    NStripe.BackgroundColor3 = C.Accent
    NStripe.BorderSizePixel = 0
    NStripe.ZIndex          = 6
    NStripe.Parent          = Notif
    Instance.new("UICorner",NStripe).CornerRadius = UDim.new(0,4)

    local NIcon = Instance.new("TextLabel")
    NIcon.Text              = "⚡"
    NIcon.Font              = Enum.Font.GothamBold
    NIcon.TextSize          = 22
    NIcon.TextColor3        = C.Accent
    NIcon.BackgroundTransparency = 1
    NIcon.Size              = UDim2.new(0,36,1,0)
    NIcon.Position          = UDim2.new(0,10,0,0)
    NIcon.ZIndex            = 6
    NIcon.Parent            = Notif

    local NTitle = Instance.new("TextLabel")
    NTitle.Text             = "Nexus Hub v1.0"
    NTitle.Font             = Enum.Font.GothamBold
    NTitle.TextSize         = 13
    NTitle.TextColor3       = C.White
    NTitle.BackgroundTransparency = 1
    NTitle.Size             = UDim2.new(1,-52,0,22)
    NTitle.Position         = UDim2.new(0,48,0,8)
    NTitle.TextXAlignment   = Enum.TextXAlignment.Left
    NTitle.ZIndex           = 6
    NTitle.Parent           = Notif

    local NSub = Instance.new("TextLabel")
    NSub.Text               = "by Minh Scripts  •  Loaded!"
    NSub.Font               = Enum.Font.Gotham
    NSub.TextSize           = 11
    NSub.TextColor3         = C.Sub
    NSub.BackgroundTransparency = 1
    NSub.Size               = UDim2.new(1,-52,0,18)
    NSub.Position           = UDim2.new(0,48,0,30)
    NSub.TextXAlignment     = Enum.TextXAlignment.Left
    NSub.ZIndex             = 6
    NSub.Parent             = Notif

    -- slide in, wait, slide out
    local function ShowNotif()
        TweenService:Create(Notif, TweenInfo.new(0.35,Enum.EasingStyle.Back,Enum.EasingDirection.Out), {
            Position = UDim2.new(1,-250,1,-80)
        }):Play()
        task.delay(3.5, function()
            TweenService:Create(Notif, TweenInfo.new(0.3,Enum.EasingStyle.Quad,Enum.EasingDirection.In), {
                Position = UDim2.new(1,20,1,-80)
            }):Play()
            task.delay(0.4, function()
                NotifGui:Destroy()
            end)
        end)
    end

    -- ════════════════════════════════════════════════
    --  ANIMATE LOADING SCREEN THEN REVEAL MAIN GUI
    -- ════════════════════════════════════════════════
    task.spawn(function()
        for _,step in ipairs(steps) do
            TweenService:Create(PBarFill, TweenInfo.new(0.35), {
                Size = UDim2.new(step[1],0,1,0)
            }):Play()
            LoadStatus.Text = step[2]
            task.wait(0.3)
        end
        task.wait(0.2)

        -- fade out loading screen
        TweenService:Create(LoadBG, TweenInfo.new(0.5), {
            BackgroundTransparency = 1
        }):Play()
        for _,c in ipairs(LoadBG:GetDescendants()) do
            if c:IsA("TextLabel") or c:IsA("Frame") then
                pcall(function()
                    TweenService:Create(c, TweenInfo.new(0.4), {
                        BackgroundTransparency = 1,
                        TextTransparency       = 1,
                    }):Play()
                end)
            end
        end

        task.wait(0.55)
        LoadGui:Destroy()

        -- reveal main GUI with a fade-in
        SG.Enabled = true
        Win.BackgroundTransparency = 1
        Shad.BackgroundTransparency = 1
        TweenService:Create(Win, TweenInfo.new(0.3), {BackgroundTransparency=0}):Play()
        TweenService:Create(Shad, TweenInfo.new(0.3), {BackgroundTransparency=0.5}):Play()

        task.wait(0.4)
        ShowNotif()
    end)

end -- end GUI do block

print("[Nexus Hub v1.0 — Minh Scripts] Loaded")
