# Baddies
-- __TP error bootstrap: runs BEFORE anything else can fail.
local __TP = nil
pcall(function()
    local g = (getgenv and getgenv()) or _G
    g.__TP = g.__TP or { log = {}, ok = false }
    __TP = g.__TP
end)
if not __TP then __TP = { log = {}, ok = false } end

local function __tpFlush()
    pcall(function()
        if writefile then writefile("TokenPanel_errors.log", table.concat(__TP.log, "\n")) end
    end)
end

local function __tpPush(kind, msg)
    pcall(function()
        local line = (os.date("%H:%M:%S") or "?") .. " [" .. tostring(kind) .. "] " .. tostring(msg):sub(1, 500)
        table.insert(__TP.log, line)
        if #__TP.log > 200 then table.remove(__TP.log, 1) end
        __tpFlush()
    end)
end

local function __tpShowViewer()
    pcall(function()
        local plrs = game:GetService("Players")
        local lp = plrs.LocalPlayer or plrs.PlayerAdded:Wait()
        local pg = lp:WaitForChild("PlayerGui")
        local old = pg:FindFirstChild("TokenPanel_Errors")
        if old then old:Destroy() end
        local sg = Instance.new("ScreenGui")
        sg.Name = "TokenPanel_Errors"
        sg.ResetOnSpawn = false
        sg.DisplayOrder = 1000
        sg.Parent = pg
        local fr = Instance.new("Frame")
        fr.Size = UDim2.new(0, 420, 0, 300)
        fr.Position = UDim2.new(0.5, -210, 0.5, -150)
        fr.BackgroundColor3 = Color3.fromRGB(20, 12, 12)
        fr.BorderSizePixel = 0
        fr.Active = true
        fr.Draggable = true
        fr.Parent = sg
        Instance.new("UICorner", fr).CornerRadius = UDim.new(0, 8)
        local tb = Instance.new("TextBox")
        tb.Size = UDim2.new(1, -20, 1, -70)
        tb.Position = UDim2.new(0, 10, 0, 10)
        tb.BackgroundColor3 = Color3.fromRGB(10, 10, 12)
        tb.TextColor3 = Color3.fromRGB(255, 170, 170)
        tb.Font = Enum.Font.Code
        tb.TextSize = 12
        tb.TextXAlignment = Enum.TextXAlignment.Left
        tb.TextYAlignment = Enum.TextYAlignment.Top
        tb.TextWrapped = true
        tb.MultiLine = true
        tb.ClearTextOnFocus = false
        tb.Text = (#__TP.log > 0) and table.concat(__TP.log, "\n") or "No errors logged yet."
        tb.Parent = fr
        local copy = Instance.new("TextButton")
        copy.Size = UDim2.new(0.5, -15, 0, 26)
        copy.Position = UDim2.new(0, 10, 1, -36)
        copy.BackgroundColor3 = Color3.fromRGB(45, 45, 58)
        copy.Font = Enum.Font.GothamBold
        copy.TextSize = 12
        copy.TextColor3 = Color3.new(1, 1, 1)
        copy.Text = "Copy"
        copy.Parent = fr
        Instance.new("UICorner", copy).CornerRadius = UDim.new(0, 6)
        copy.MouseButton1Click:Connect(function()
            pcall(function() setclipboard(tb.Text) copy.Text = "Copied!" end)
        end)
        local close = Instance.new("TextButton")
        close.Size = UDim2.new(0.5, -15, 0, 26)
        close.Position = UDim2.new(0.5, 5, 1, -36)
        close.BackgroundColor3 = Color3.fromRGB(45, 45, 58)
        close.Font = Enum.Font.GothamBold
        close.TextSize = 12
        close.TextColor3 = Color3.new(1, 1, 1)
        close.Text = "Close"
        close.Parent = fr
        Instance.new("UICorner", close).CornerRadius = UDim.new(0, 6)
        close.MouseButton1Click:Connect(function() sg:Destroy() end)
    end)
end

local __tpOK, __tpErr = xpcall(function()
-- ============================================================
-- BADDIES GUI â FULL
-- ============================================================

local Players           = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local SoundService      = game:GetService("SoundService")
local TweenService      = game:GetService("TweenService")
local Workspace         = game:GetService("Workspace")
local Debris            = game:GetService("Debris")
local UIS               = game:GetService("UserInputService")
local HttpService       = game:GetService("HttpService")
local CollectionService = game:GetService("CollectionService")
local StarterGui        = game:GetService("StarterGui")

local LocalPlayer = Players.LocalPlayer or Players.PlayerAdded:Wait()
local PlayerGui   = LocalPlayer:WaitForChild("PlayerGui")

local ADMIN = "baddiesplusthiscat"
local isAdmin = (LocalPlayer.Name:lower() == ADMIN:lower())

-- ============================================================
-- MODULE LOADING
-- ============================================================
local Modules = ReplicatedStorage:FindFirstChild("Modules")
local Replion = {}
local WeaponsData = {}
local FilteredWeapons = {}

pcall(function()
    local m = Modules and Modules:FindFirstChild("Replion")
    if m then Replion = require(m) end
end)
pcall(function()
    local Shared = ReplicatedStorage:FindFirstChild("Shared")
    local w = Shared and Shared:FindFirstChild("Weapons")
    if w then
        WeaponsData = require(w)
        for name, info in pairs(WeaponsData) do
            local skip = false
            if info.Tradeable == false then skip = true end
            if info.Temporary == true then skip = true end
            if not skip then FilteredWeapons[name] = info end
        end
    end
end)

local Client = Replion.Client

-- ============================================================
-- GIVER OPTIONS
-- ============================================================
local TOKEN_OPTIONS = {
    { Amount = 10000,   Wait = 2*60,  Label = "10K Tokens  â¢  2m" },
    { Amount = 50000,   Wait = 5*60,  Label = "50K Tokens  â¢  5m" },
    { Amount = 100000,  Wait = 8*60,  Label = "100K Tokens  â¢  8m" },
    { Amount = 1000000, Wait = 10*60, Label = "1M Tokens  â¢  10m" },
}
local DINERO_OPTIONS = {
    { Amount = 100000,   Wait = 2*60,  Label = "100K Dinero  â¢  2m" },
    { Amount = 500000,   Wait = 5*60,  Label = "500K Dinero  â¢  5m" },
    { Amount = 1000000,  Wait = 8*60,  Label = "1M Dinero  â¢  8m" },
    { Amount = 10000000, Wait = 10*60, Label = "10M Dinero  â¢  10m" },
}
local SLAY_OPTIONS = {
    { Amount = 100,  Wait = 2*60,  Label = "100 Slays  â¢  2m" },
    { Amount = 500,  Wait = 5*60,  Label = "500 Slays  â¢  5m" },
    { Amount = 1000, Wait = 8*60,  Label = "1K Slays  â¢  8m" },
    { Amount = 5000, Wait = 10*60, Label = "5K Slays  â¢  10m" },
}

-- ============================================================
-- STATE
-- ============================================================
local net         = 0
local netDinero   = 0
local netSlays    = 0
local baseTokens  = 0
local baseDinero  = 0
local baseSlays   = 0

local activeJob    = { token = nil, dinero = nil, slay = nil }
local statusLabels = { token = nil, dinero = nil, slay = nil }

local fakeSpins   = 0
local fakeHourly  = 0
local fakeTickets = 0
local fakeHSpins  = 0
local fakeEmote   = 0
local fakeGacha   = 0
local fakeRaffle  = 0

local SPIN_COST   = 2500
local SPIN1       = 2500
local SPIN3       = 6000
local SPIN10      = 15000
local H_SPIN      = 2500
local H_BUY3      = 6000
local H_BUY10     = 15000
local F_TICKET    = 2500
local H_SPIN_COST = 2500
local H_BUY10_COST= 15000
local E_SPIN1     = 2500
local E_SPIN10    = 15000
local G_SPIN1     = 2500
local G_SPIN10    = 15000
local R_SPIN1     = 2500
local R_SPIN10    = 15000

-- ============================================================
-- LEADERSTAT / TOKEN READS
-- ============================================================
local function readLeaderstat(name)
    local n = 0
    pcall(function()
        local ls = LocalPlayer:FindFirstChild("leaderstats")
        if ls then
            local v = ls:FindFirstChild(name)
            if v then n = tonumber(v.Value) or tonumber((v.Text or ""):gsub("[^%d]", "")) or 0 end
        end
    end)
    return n
end

local function readTokens()
    local n = 0
    pcall(function()
        if Client then
            local Data = Client:GetReplion("Data")
            if Data then n = tonumber(Data:Get("NewInventory.Tokens")) or 0 end
        end
    end)
    if n == 0 then
        pcall(function()
            local tl = PlayerGui:FindFirstChild("TokenStore", 5)
            tl = tl and tl:FindFirstChild("Main", 5)
            tl = tl and tl:FindFirstChild("Main", 5)
            tl = tl and tl:FindFirstChild("TokenAmount", 5)
            tl = tl and tl:FindFirstChild("TextLabel", 5)
            if tl then n = tonumber((tl.Text:gsub("[^%d]", ""))) or 0 end
        end)
    end
    return n
end

local DataRef = nil
pcall(function()
    if Client then DataRef = Client:GetReplion("Data") end
end)

baseTokens = readTokens()
baseDinero = readLeaderstat("Dinero")
baseSlays  = readLeaderstat("Slays")

task.spawn(function()
    while true do
        task.wait(5)
        local t = readTokens();              if t > 0 then baseTokens = t end
        local d = readLeaderstat("Dinero");  if d > 0 then baseDinero = d end
        local s = readLeaderstat("Slays");   if s > 0 then baseSlays  = s end
    end
end)

local function shownTokens() return math.max(0, baseTokens + net) end
local function shownDinero() return math.max(0, baseDinero + netDinero) end
local function shownSlays()  return math.max(0, baseSlays  + netSlays)  end

-- ============================================================
-- FORMATTERS
-- ============================================================
local function fmtRaw(n) return tostring(math.floor(tonumber(n) or 0)) end
local function fmtCommas(n)
    n = math.floor(tonumber(n) or 0)
    local s = tostring(math.abs(n))
    local out, c = "", 0
    for i = #s, 1, -1 do
        out = s:sub(i,i) .. out
        c = c + 1
        if c % 3 == 0 and i > 1 then out = "," .. out end
    end
    if n < 0 then out = "-" .. out end
    return out
end
local function fmtTime(sec)
    sec = math.max(0, math.floor(sec + 0.5))
    return string.format("%d:%02d", math.floor(sec / 60), sec % 60)
end
local function capitalizeFirstLetter(s)
    return s:gsub("(%a)([%w_\']*)", function(a,b) return a:upper() .. b:lower() end)
end

-- ============================================================
-- SOUNDS / NOTIFICATIONS
-- ============================================================
local GameMessage = nil
pcall(function()
    GameMessage = require(ReplicatedStorage:WaitForChild("Modules", 5):WaitForChild("Message", 5))
end)

local function rewardSound()
    local ok, s = pcall(function()
        local w = ReplicatedStorage:FindFirstChild("SpinWheel", true)
        if w then
            local f = w:FindFirstChild("Sounds")
            if f then return f:FindFirstChild("reward") or f:FindFirstChild("spin") end
        end
    end)
    if ok and s and s:IsA("Sound") then return s end
    local ok2, s2 = pcall(function()
        return SoundService:WaitForChild("Sounds", 5):WaitForChild("WheelReward", 5)
    end)
    if ok2 and s2 then return s2 end
    return nil
end

local function spinSound()
    local ok, s = pcall(function()
        local w = ReplicatedStorage:FindFirstChild("SpinWheel", true)
        if w then
            local f = w:FindFirstChild("Sounds")
            if f then return f:FindFirstChild("spin") end
        end
    end)
    if ok and s and s:IsA("Sound") then return s end
    return rewardSound()
end

local function tickSound()
    local ok, s = pcall(function()
        local w = ReplicatedStorage:FindFirstChild("SpinWheel", true)
        if w then
            local f = w:FindFirstChild("Sounds")
            if f then return f:FindFirstChild("tick") end
        end
    end)
    if ok and s and s:IsA("Sound") then return s end
    return nil
end

local function playLocal(s)
    if s then pcall(function() SoundService:PlayLocalSound(s) end); return end
    pcall(function()
        local t = Instance.new("Sound")
        t.SoundId = "rbxassetid://4612378086"
        t.Parent = PlayerGui
        t:Play()
        Debris:AddItem(t, 4)
    end)
end

local function playTick()
    pcall(function() local t = tickSound(); if t then SoundService:PlayLocalSound(t) end end)
end

local suppressUntil = 0
local inNotify = false
pcall(function()
    if GameMessage then
        local orig = GameMessage.Message
        if type(orig) == "function" then
            GameMessage.Message = function(text, snd)
                if not inNotify and os.clock() < suppressUntil then return nil end
                return orig(text, snd)
            end
        end
    end
end)

local fbGui, fbLabel, fbUntil = nil, nil, 0
local function fallbackBanner(text)
    pcall(function()
        if not fbGui or not fbGui.Parent then
            fbGui = Instance.new("ScreenGui")
            fbGui.Name = "TokenPanel_Note"
            fbGui.ResetOnSpawn = false
            fbGui.DisplayOrder = 999
            fbGui.Parent = PlayerGui
            local fr = Instance.new("Frame")
            fr.AnchorPoint = Vector2.new(0.5, 0)
            fr.Position = UDim2.new(0.5, 0, 0, 70)
            fr.Size = UDim2.new(0, 560, 0, 44)
            fr.BackgroundColor3 = Color3.fromRGB(0,0,0)
            fr.BackgroundTransparency = 0.35
            fr.BorderSizePixel = 0
            fr.Parent = fbGui
            Instance.new("UICorner", fr).CornerRadius = UDim.new(0, 8)
            fbLabel = Instance.new("TextLabel")
            fbLabel.Size = UDim2.new(1, -16, 1, -4)
            fbLabel.Position = UDim2.new(0, 8, 0, 2)
            fbLabel.BackgroundTransparency = 1
            fbLabel.Font = Enum.Font.GothamBlack
            fbLabel.TextSize = 17
            fbLabel.TextWrapped = true
            fbLabel.TextColor3 = Color3.fromRGB(255,235,60)
            fbLabel.TextStrokeTransparency = 0.2
            fbLabel.Parent = fr
        end
        fbLabel.Text = text
        fbGui.Enabled = true
        fbUntil = os.clock() + 4
        task.delay(4.1, function()
            if os.clock() >= fbUntil - 0.05 then pcall(function() fbGui.Enabled = false end) end
        end)
    end)
end

local Rayfield = nil

local function notify(text, withSound)
    inNotify = true
    local delivered = false
    pcall(function()
        if GameMessage and GameMessage.Message then
            if withSound == false then
                GameMessage.Message(text)
            else
                local ok = pcall(function() GameMessage.Message(text, rewardSound()) end)
                if not ok then GameMessage.Message(text) end
            end
            delivered = true
        end
    end)
    inNotify = false
    if not delivered then fallbackBanner(text) end
    if withSound ~= false then playLocal(rewardSound()) end
end

-- ============================================================
-- HUD SPOOF
-- ============================================================
local function spoofHUD()
    pcall(function()
        local total = shownTokens()
        if net ~= 0 then
            local tl = PlayerGui:FindFirstChild("TokenStore")
            tl = tl and tl:FindFirstChild("Main")
            tl = tl and tl:FindFirstChild("Main")
            tl = tl and tl:FindFirstChild("TokenAmount")
            tl = tl and tl:FindFirstChild("TextLabel")
            if tl and tl:IsA("TextLabel") then tl.Text = "Tokens: " .. fmtRaw(total) end
            local t2 = PlayerGui:FindFirstChild("TradeList")
            t2 = t2 and t2:FindFirstChild("Main")
            t2 = t2 and t2:FindFirstChild("TokenAmount")
            t2 = t2 and t2:FindFirstChild("TextLabel")
            if t2 and t2:IsA("TextLabel") then t2.Text = "Tokens: " .. fmtRaw(total) end
            local tp = PlayerGui:FindFirstChild("TokenPopup")
            local buy = tp and tp:FindFirstChild("BuyItem")
            local cur = buy and buy:FindFirstChild("Currency")
            if cur then
                for _, d in ipairs(cur:GetDescendants()) do
                    if d:IsA("TextLabel") and d.Name == "Total" then d.Text = fmtRaw(total) end
                end
            end
        end
    end)
    pcall(function()
        local ls = LocalPlayer:FindFirstChild("leaderstats")
        if ls then
            if netDinero ~= 0 then
                local d = ls:FindFirstChild("Dinero"); if d then d.Value = shownDinero() end
            end
            if netSlays ~= 0 then
                local s = ls:FindFirstChild("Slays"); if s then s.Value = shownSlays() end
            end
        end
    end)
end

task.spawn(function()
    while true do pcall(spoofHUD); task.wait(0.3) end
end)

-- ============================================================
-- WEAPON ADMIN
-- ============================================================
local activeCooldowns = {}
local COOLDOWN_SECONDS = 30

local function alreadyOwns(weaponName)
    if not Client then return false end
    local ok, data = pcall(function() return Client:GetReplion("Data") end)
    if not ok or not data then return false end
    local ok2, inv = pcall(function() return data:Get("NewInventory.Items") end)
    if not ok2 or not inv then return false end
    for _, list in pairs(inv) do
        for _, item in pairs(list) do
            if item and item.Name == weaponName then return true end
        end
    end
    return false
end

local function addWeaponTags(tool, weaponName, weaponInfo)
    local rawName = weaponName
    local toolName = weaponInfo.ToolName or weaponName
    local variants = {
        rawName, rawName:lower(), rawName:gsub("%s+", ""), rawName:gsub("%s+", ""):lower(),
        toolName, toolName:lower(), toolName:gsub("%s+", ""), toolName:gsub("%s+", ""):lower(),
    }
    for _, tag in ipairs(variants) do
        if tag ~= "" then
            pcall(function() CollectionService:AddTag(tool, tag) end)
        end
    end
end

local function AdminGiveWeapon(weaponName, silent)
    if not Client then
        if not silent then notify("Client not available", false) end
        return false
    end
    local weaponInfo = FilteredWeapons[weaponName]
    if not weaponInfo then
        if not silent then
            local info = WeaponsData[weaponName]
            if info and info.Tradeable == false then notify("Untradeable: " .. weaponName, false)
            elseif info and info.Temporary == true then notify("Temporary: " .. weaponName, false)
            else notify("Unknown weapon: " .. weaponName, false) end
        end
        return false
    end
    if alreadyOwns(weaponName) then
        if not silent then notify("Already own: " .. weaponName, false) end
        return false
    end
    local ok2, dataReplion = pcall(function() return Client:GetReplion("Data") end)
    if not ok2 or not dataReplion then
        if not silent then notify("Cannot access data", false) end
        return false
    end
    local newId = "ADMIN_" .. HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 8)
    local newItem = { Name = weaponName, Id = newId, Equipped = false }
    local ok3 = pcall(function()
        dataReplion:_set("NewInventory.Items.Weapon." .. newId, newItem)
    end)
    if not ok3 then
        if not silent then notify("Failed to set data", false) end
        return false
    end

    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if not backpack then return false end
    local toolName = weaponInfo.ToolName or weaponName
    local toolStorage = ReplicatedStorage:FindFirstChild("exe_storage") and ReplicatedStorage.exe_storage:FindFirstChild("tools")
    local realTool = nil
    if toolStorage then
        for _, child in pairs(toolStorage:GetChildren()) do
            if child:IsA("Tool") then
                local ctn = child:GetAttribute("ToolName")
                if ctn == toolName or child.Name == toolName or child.Name:lower() == toolName:lower() then
                    realTool = child; break
                end
            end
        end
        if not realTool then
            for _, child in pairs(toolStorage:GetChildren()) do
                if child:IsA("Tool") and child.Name:lower() == weaponName:lower() then realTool = child; break end
            end
        end
    end
    local tool
    if realTool then
        tool = realTool:Clone()
        tool.Parent = backpack
    else
        tool = Instance.new("Tool")
        tool.Name = weaponName
        tool.TextureId = weaponInfo.Icon or ""
        tool.RequiresHandle = true
        tool.Parent = backpack
        tool:SetAttribute("AdminTemp", true)
        tool:SetAttribute("OriginalWeapon", weaponName)
        addWeaponTags(tool, weaponName, weaponInfo)
        local handle = Instance.new("Part")
        handle.Name = "Handle"
        handle.Size = Vector3.new(0.5, 0.5, 3)
        handle.Transparency = 1
        handle.CanCollide = false
        handle.Parent = tool
    end
    addWeaponTags(tool, weaponName, weaponInfo)
    if not tool:GetAttribute("AdminTemp") then tool:SetAttribute("AdminTemp", true) end
    if not tool:GetAttribute("OriginalWeapon") then tool:SetAttribute("OriginalWeapon", weaponName) end
    if not silent then notify("Gave: " .. weaponName, false) end
    return true
end

local function AdminDuplicateItems()
    if not Client then return end
    local ok, dataReplion = pcall(function() return Client:GetReplion("Data") end)
    if not ok or not dataReplion then return end
    local ok2, inventory = pcall(function() return dataReplion:Get("NewInventory.Items") end)
    if not ok2 or not inventory then return end
    local duplicated, skipped = 0, 0
    for category, items in pairs(inventory) do
        local keys = {}
        for id, _ in pairs(items) do table.insert(keys, id) end
        for _, id in ipairs(keys) do
            local itemData = items[id]
            if itemData then
                local info = WeaponsData[itemData.Name]
                if info and (info.Tradeable == false or info.Temporary == true) then
                    skipped = skipped + 1
                else
                    local newId = "DUP_" .. HttpService:GenerateGUID(false):gsub("-", ""):sub(1, 8)
                    local newItem = {}
                    for k, v in pairs(itemData) do newItem[k] = v end
                    newItem.Id = newId
                    local ok3 = pcall(function()
                        dataReplion:_set("NewInventory.Items." .. category .. "." .. newId, newItem)
                    end)
                    if ok3 then duplicated = duplicated + 1 end
                end
            end
        end
    end
    notify("Duplicated " .. duplicated .. " (skipped " .. skipped .. ")", true)
end

LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.5)
    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        for _, tool in pairs(bp:GetChildren()) do
            if tool:IsA("Tool") and tool:GetAttribute("AdminTemp") then tool:Destroy() end
        end
    end
    local char = LocalPlayer.Character
    if char then
        for _, tool in pairs(char:GetChildren()) do
            if tool:IsA("Tool") and tool:GetAttribute("AdminTemp") then tool:Destroy() end
        end
    end
end)

-- ============================================================
-- WHEEL DATA LOADING
-- ============================================================
local WheelOdds, WheelNames = nil, nil
local WheelAnimT = 4
local WheelAnimStyle = Enum.EasingStyle.Quart
local WheelAnimDir = Enum.EasingDirection.Out
local HourlyCrates, HourlyAnimT = nil, 4
local HourlyAnimStyle = Enum.EasingStyle.Quart
local HourlyAnimDir = Enum.EasingDirection.Out
local FestCrates, HaloMod = nil, nil
local GachaRewards, RaffleRewards, EmoteRewards = nil, nil, nil

pcall(function()
    local st = require(ReplicatedStorage:WaitForChild("SpinWheel", 5):WaitForChild("Settings", 5))
    if type(st) == "table" then
        if st.Odds then WheelOdds = st.Odds end
        if st.DisplayNames then WheelNames = st.DisplayNames end
        if type(st.AnimationTime) == "number" then WheelAnimT = st.AnimationTime end
        if st.AnimationStyle then WheelAnimStyle = st.AnimationStyle end
        if st.AnimationDirection then WheelAnimDir = st.AnimationDirection end
    end
end)
pcall(function()
    local st = require(ReplicatedStorage:WaitForChild("Shared", 5):WaitForChild("HourlySpin", 5):WaitForChild("Settings", 5))
    if type(st) == "table" then
        if st.Crates then HourlyCrates = st.Crates end
        if type(st.AnimationTime) == "number" then HourlyAnimT = st.AnimationTime end
        if st.AnimationStyle then HourlyAnimStyle = st.AnimationStyle end
        if st.AnimationDirection then HourlyAnimDir = st.AnimationDirection end
    end
end)
pcall(function()
    local sh = ReplicatedStorage:WaitForChild("Shared", 5)
    local fm = sh:FindFirstChild("FestivalSpinWheel") or sh:FindFirstChild("Festival")
    local st = fm and (fm:FindFirstChild("Settings") or fm)
    local m = st and require(st)
    if type(m) == "table" and m.Crates then FestCrates = m.Crates end
end)
pcall(function()
    HaloMod = require(ReplicatedStorage:WaitForChild("Shared", 5):WaitForChild("WitchingHour", 5):WaitForChild("HalloweenSpin", 5))
end)
pcall(function()
    local Net = require(ReplicatedStorage:WaitForChild("Modules", 5):WaitForChild("Net", 5))
    local a = Net:Invoke("Gacha_GetRewards");       if a then GachaRewards = a end
    local c = Net:Invoke("RaffleGacha_GetRewards"); if c then RaffleRewards = c end
    local e = Net:Invoke("EmotesGacha_GetRewards"); if e then EmoteRewards = e end
end)

local function luckNow()
    local l = 1
    pcall(function()
        local v = ReplicatedStorage:GetAttribute("ServerLuck")
        if type(v) == "number" and v >= 1 then l = v end
    end)
    return l
end

local function adjustOdds(map)
    local luck = luckNow()
    local keys = {}
    for k in pairs(map) do keys[#keys+1] = k end
    if luck == 1 or #keys == 0 then
        local c = {}
        for k, v in pairs(map) do c[k] = v end
        return c
    end
    local inv = 1 / luck
    local shaped, sum = {}, 0
    for k, v in pairs(map) do
        local nv = 0
        if (v or 0) > 0 then
            nv = math.floor((v ^ inv) * 10 + 0.5) / 10
            if nv == 0 then nv = 0.1 end
        end
        shaped[k] = nv; sum = sum + nv
    end
    if sum <= 0 then
        local c = {}
        for k, v in pairs(map) do c[k] = v end
        return c
    end
    local out, total = {}, 0
    local topK, topV = nil, -1/0
    for k, v in pairs(shaped) do
        local pct = v / sum * 100
        if pct > topV then topV = pct; topK = k end
        local r = math.floor(pct * 100 + 0.5) / 100
        out[k] = r; total = total + r
    end
    local rest = math.floor((100 - total) * 100 + 0.5) / 100
    if math.abs(rest) >= 0.01 and topK ~= nil then
        out[topK] = out[topK] + rest
        if out[topK] < 0 then out[topK] = 0 end
    end
    return out
end

local function rollEntries(entries)
    if #entries == 0 then return nil end
    if #entries == 1 then return entries[1] end
    local map = {}
    for _, e in ipairs(entries) do map[e.key] = e.chance or 0 end
    local adj = adjustOdds(map)
    local total = 0
    for _, e in ipairs(entries) do total = total + (adj[e.key] or 0) end
    if total <= 0 then return entries[math.random(1, #entries)] end
    local r = math.random() * total
    local acc = 0
    for _, e in ipairs(entries) do
        acc = acc + (adj[e.key] or 0)
        if r <= acc then return e end
    end
    return entries[#entries]
end

local function flatDict(dict)
    local out = {}
    if type(dict) == "table" then
        for k, v in pairs(dict) do
            if type(v) == "table" and v.Name then
                out[#out+1] = { key = tostring(k), name = v.Name, chance = tonumber(v.Chance) or 0 }
            end
        end
    end
    return out
end

local function flatNested(dict)
    local out = {}
    if type(dict) == "table" then
        for cat, sub in pairs(dict) do
            if type(sub) == "table" and cat ~= "SpinnedTimes" then
                for k, v in pairs(sub) do
                    if type(v) == "table" and v.Name then
                        out[#out+1] = { key = tostring(k), cat = tostring(cat), name = v.Name, chance = tonumber(v.Chance) or 0 }
                    end
                end
            end
        end
    end
    return out
end

local function tweenToSeg(spinner, seg, animT, animStyle, animDir)
    if not spinner then return end
    pcall(function()
        playLocal(spinSound())
        local target = 1800 + math.random(0, 360)
        if seg then
            local r = seg.Rotation
            if type(r) == "number" then target = -(r + math.random(-20, 20)) + 1800 end
        end
        spinner.Rotation = 0
        TweenService:Create(spinner,
            TweenInfo.new(animT or 4, animStyle or Enum.EasingStyle.Quart, animDir or Enum.EasingDirection.Out),
            { Rotation = target }):Play()
    end)
end

local function highlightRun(frames, target)
    if #frames == 0 then return target end
    local delay = 0.06
    for step = 1, 14 do
        local f = frames[((step - 1) % #frames) + 1]
        pcall(function()
            local sc = Instance.new("UIScale"); sc.Scale = 1.2; sc.Parent = f
            Debris:AddItem(sc, delay + 0.05)
            local ar = f:FindFirstChild("Arrow")
            if ar then
                ar.Visible = true
                task.delay(delay, function() pcall(function() ar.Visible = false end) end)
            end
        end)
        playTick()
        task.wait(delay)
        delay = math.min(delay * 1.22, 0.35)
    end
    local land = target
    if not land or not land.Parent then land = frames[math.random(1, #frames)] end
    pcall(function()
        local sc = Instance.new("UIScale"); sc.Scale = 1.2; sc.Parent = land
        local ar = land:FindFirstChild("Arrow")
        if ar then ar.Visible = true end
        task.delay(2.5, function()
            pcall(function()
                sc:Destroy()
                if ar then ar.Visible = false end
            end)
        end)
    end)
    playLocal(rewardSound())
    return land
end

local function liveLabel(frame)
    local nm = nil
    pcall(function()
        if frame then
            local il = frame:FindFirstChild("ItemName")
            if il and il:IsA("TextLabel") and il.Text ~= "" then nm = il.Text end
            if not nm then
                local tt = frame:FindFirstChild("Title")
                if tt and tt:IsA("TextLabel") and tt.Text ~= "" then nm = tt.Text end
            end
        end
    end)
    if nm then nm = nm:gsub("%s+%d+%.?%d*%%$", "") end
    return nm
end

local function pickRewardFromUI(root)
    if not root then return nil end
    local cands = {}
    pcall(function()
        for _, d in ipairs(root:GetDescendants()) do
            if d:IsA("TextLabel") and (d.Name == "ItemName" or d.Name == "Title") then
                local t = d.Text
                if t and #t > 0 and #t < 80 then
                    if not t:match("^%d+%.?%d*%%$") then
                        cands[#cands+1] = t
                    else
                        local nm = t:gsub("%s+%d+%.?%d*%%$", "")
                        if #nm > 0 then cands[#cands+1] = nm end
                    end
                end
            end
        end
    end)
    if #cands == 0 then return nil end
    return cands[math.random(1, #cands)]
end

local function disconnectGame(btn)
    if not btn then return end
    pcall(function()
        local gc = (getconnections or get_connections)
        if gc and btn.MouseButton1Click then
            for _, c in ipairs(gc(btn.MouseButton1Click)) do
                if c.Disable then c:Disable() elseif c.Disconnect then c:Disconnect() end
            end
        end
        if gc and btn.Activated then
            for _, c in ipairs(gc(btn.Activated)) do
                if c.Disable then c:Disable() elseif c.Disconnect then c:Disconnect() end
            end
        end
    end)
end

-- ============================================================
-- POPUP (game Robux/Tokens choice)
-- ============================================================
local Market = nil
pcall(function()
    Market = require(LocalPlayer:WaitForChild("PlayerScripts", 10)
        :WaitForChild("Controllers", 10)
        :WaitForChild("MarketplaceController", 10))
end)

local function canBuyGate(rfName)
    local allowed = true
    pcall(function()
        local Net = require(ReplicatedStorage:WaitForChild("Modules", 5):WaitForChild("Net", 5))
        if Net then
            local rf = Net:RemoteFunction(rfName)
            if rf then
                local ok, msg = rf:InvokeServer()
                if ok == false then allowed = false; notify(msg or "Not available", false) end
            end
        end
    end)
    return allowed
end

local popupWired = false
local popupTokensCb = nil
local popupProduct = 0
local popupTokenPrice = 0
local popupKind = "Product"
local popupMode = nil

local function popupParts()
    local ok, res = pcall(function()
        local tp = PlayerGui:WaitForChild("TokenPopup", 3)
        local buy = tp:WaitForChild("BuyItem", 3)
        local function findTotal(btn)
            local found
            pcall(function()
                if btn then
                    for _, d in ipairs(btn:GetDescendants()) do
                        if d:IsA("TextLabel") and d.Name == "Total" then found = d; break end
                    end
                end
            end)
            return found
        end
        local tBtn = buy:WaitForChild("Tokens", 3)
        local rBtn = buy:WaitForChild("Robux", 3)
        return {
            root = tp, buy = buy,
            tokensBtn = tBtn, robuxBtn = rBtn,
            tokensTotal = findTotal(tBtn), robuxTotal = findTotal(rBtn)
        }
    end)
    if ok and res and res.tokensBtn and res.robuxBtn then return res end
end

local function openChoice(productId, fallbackTokens, onTokens, kind, mode)
    kind = kind or "Product"
    if type(productId) ~= "number" then productId = 0 end
    if type(fallbackTokens) ~= "number" then fallbackTokens = 0 end
    popupProduct = productId
    popupTokensCb = onTokens
    popupTokenPrice = fallbackTokens
    popupKind = kind
    popupMode = mode

    local tok = fallbackTokens
    popupTokenPrice = tok
    local p = popupParts()
    if not p then
        if mode ~= "soon" then
            if net < tok then notify("Need " .. fmtRaw(tok) .. " Tokens", false); return end
            net = net - tok
            pcall(spoofHUD)
            onTokens()
        else
            notify("Soon...", false)
        end
        return
    end

    pcall(function()
        if mode == "soon" then
            if p.tokensTotal then p.tokensTotal.Text = "Soon..." end
        else
            if p.tokensTotal then p.tokensTotal.Text = fmtRaw(tok) end
        end
        if p.robuxTotal then p.robuxTotal.Text = "? Robux" end
        p.buy.Tokens.Visible = true
        p.root.Enabled = true
    end)

    pcall(function()
        local tokensFrame = p.buy:FindFirstChild("Tokens")
        if tokensFrame then
            local have = tokensFrame:FindFirstChild("YouHave")
            if not have then
                have = Instance.new("TextLabel")
                have.Name = "YouHave"
                have.BackgroundTransparency = 1
                have.Font = Enum.Font.Gotham
                have.TextSize = 12
                have.TextColor3 = Color3.fromRGB(180, 220, 255)
                have.TextXAlignment = Enum.TextXAlignment.Center
                have.Size = UDim2.new(1, 0, 0, 14)
                have.Position = UDim2.new(0, 0, 1, -18)
                have.Parent = tokensFrame
            end
            have.Text = "You have: " .. fmtRaw(shownTokens()) .. " Tokens"
        end
    end)

    pcall(function()
        local cur = p.buy:FindFirstChild("Currency")
        if cur then
            for _, d in ipairs(cur:GetDescendants()) do
                if d:IsA("TextLabel") and d.Name == "Total" then d.Text = fmtRaw(shownTokens()) end
            end
        end
    end)

    if not popupWired then
        popupWired = true
        disconnectGame(p.tokensBtn)
        disconnectGame(p.robuxBtn)
        pcall(function()
            p.tokensBtn.Activated:Connect(function()
                local cb = popupTokensCb
                local price = popupTokenPrice
                local md = popupMode
                p.root.Enabled = false
                if md == "soon" then notify("Soon...", false); return end
                if not cb then return end
                if net < price then notify("Need " .. fmtRaw(price) .. " Tokens", false); return end
                net = net - price
                pcall(spoofHUD)
                cb()
            end)
            p.robuxBtn.Activated:Connect(function()
                local pid = popupProduct
                local kd = popupKind
                p.root.Enabled = false
                if type(pid) ~= "number" or pid == 0 then notify("Robux unavailable", false); return end
                pcall(function()
                    local ms = game:GetService("MarketplaceService")
                    if kd == "GamePass" then ms:PromptGamePassPurchase(LocalPlayer, pid)
                    else ms:PromptProductPurchase(LocalPlayer, pid) end
                end)
            end)
        end)
    end
end

-- ============================================================
-- PRODUCT IDS
-- ============================================================
local PID = {
    wheel1 = 1913723900, wheel1Offer = 0, wheel3 = 0, wheel10 = 1913723901,
    hourly3 = 3247636553, hourly10 = 3247636687,
    halo1 = 0, halo10 = 0,
    emote1 = 0, emote10 = 0,
    gacha1 = 1908135951, gacha10 = 1908136324,
    raffle1 = 0, raffle10 = 0,
}
pcall(function()
    local s = require(ReplicatedStorage:WaitForChild("SpinWheel", 5):WaitForChild("Settings", 5))
    PID.wheel1, PID.wheel1Offer, PID.wheel3, PID.wheel10 =
        s.Spin1WithoutOffer, s.Spin1WithOffer, s.Spin3, s.Spin10
end)
pcall(function()
    local s = require(ReplicatedStorage:WaitForChild("Shared", 5):WaitForChild("HourlySpin", 5):WaitForChild("Settings", 5))
    PID.hourly3, PID.hourly10 = s.ThreeSpinProductId, s.TenSpinProductId
end)
pcall(function()
    local s = require(ReplicatedStorage:WaitForChild("Shared", 5):WaitForChild("WitchingHour", 5):WaitForChild("HalloweenSpin", 5))
    PID.halo1, PID.halo10 = s.OneSpinProductId, s.TenSpinProductId
end)
PID.hourly3 = 3247636553
PID.hourly10 = 3247636687
PID.gacha1 = 1908135951
PID.gacha10 = 1908136324
PID.wheel1 = 1913723900
PID.wheel10 = 1913723901

local Catalog = {}
task.spawn(function()
    for _ = 1, 15 do
        if PID.wheel1 ~= 0 and PID.hourly3 ~= 0 then break end
        task.wait(1)
    end
    for pass = 1, 2 do
        local ids = {}
        for _, k in ipairs({"wheel1","wheel1Offer","wheel3","wheel10","hourly3","hourly10","halo1","halo10","emote1","emote10","gacha1","gacha10","raffle1","raffle10"}) do
            local id = PID[k]
            if type(id) == "number" and id ~= 0 and not Catalog[id] then ids[id] = true end
        end
        for pid in pairs(ids) do
            for _ = 1, 6 do
                local done = false
                pcall(function()
                    local info = game:GetService("MarketplaceService"):GetProductInfo(pid, Enum.InfoType.Product)
                    if info and type(info.PriceInRobux) == "number" then
                        Catalog[pid] = { rbx = info.PriceInRobux, tok = info.PriceInRobux }
                        done = true
                    end
                end)
                if done then break end
                task.wait(1)
            end
        end
        if pass == 1 then task.wait(14) end
    end
end)

local function priceOf(buyBtn)
    local n = nil
    pcall(function()
        if buyBtn then
            local pl = buyBtn:FindFirstChild("Price")
            if not pl then
                for _, d in ipairs(buyBtn:GetDescendants()) do
                    if (d:IsA("TextLabel") or d:IsA("TextButton")) and d.Name == "Price" then pl = d; break end
                end
            end
            if pl and (pl:IsA("TextLabel") or pl:IsA("TextButton")) then
                local digits = tostring(pl.Text):gsub("[^%d]", "")
                if #digits > 0 then n = tonumber(digits) end
            end
        end
    end)
    return n
end

local function dineroAmount(key, label)
    local amt = nil
    pcall(function()
        local disp = nil
        if WheelNames and WheelNames[key] then disp = WheelNames[key] end
        local src = disp or label or ""
        local isMoney = false
        if tostring(src):find("%$") then isMoney = true end
        if key and tostring(key):find("^Money_") then isMoney = true end
        if not isMoney then return end
        local digits = tostring(src):gsub("[^%d]", "")
        if #digits > 0 then amt = tonumber(digits) end
        if not amt and key then
            local m = tostring(key):match("Money_(%d+)")
            if m then amt = tonumber(m) * 100 end
        end
    end)
    return amt
end

-- ============================================================
-- WHEEL PLAY FUNCTIONS
-- ============================================================
local function findHourlyMain()
    local m
    pcall(function()
        local hw = PlayerGui:FindFirstChild("HourlyWheel")
        m = hw and hw:WaitForChild("Main", 3)
    end)
    return m
end

local function playHourlySpin(mainGui)
    local Main = mainGui or findHourlyMain()
    if not Main then return false end
    if fakeHourly <= 0 then return false end
    fakeHourly = fakeHourly - 1
    suppressUntil = os.clock() + 1.5
    local entries = {}
    pcall(function()
        if HourlyCrates and HourlyCrates.Basic then
            for i, it in pairs(HourlyCrates.Basic) do
                if type(it) == "table" and it.Name then
                    entries[#entries+1] = { key = tostring(i), name = it.Name, chance = tonumber(it.Chance) or 0 }
                end
            end
        end
    end)
    table.sort(entries, function(a,b) return (tonumber(a.key) or 0) < (tonumber(b.key) or 0) end)
    local pick = rollEntries(entries)
    local spinner = Main:FindFirstChild("Spinner")
    local seg
    if spinner then
        if pick then pcall(function() seg = spinner:FindFirstChild(pick.key) end) end
        if not seg then
            pcall(function()
                local kids = {}
                for _, ch in ipairs(spinner:GetChildren()) do
                    if ch:IsA("GuiObject") then kids[#kids+1] = ch end
                end
                if #kids > 0 then seg = kids[math.random(1, #kids)] end
            end)
        end
        tweenToSeg(spinner, seg, HourlyAnimT, HourlyAnimStyle, HourlyAnimDir)
    else
        playLocal(spinSound())
    end
    pcall(function()
        local sm = Workspace:FindFirstChild("SpinWheelModel")
        local rot = sm and sm:FindFirstChild("RotatingPart", true)
        if rot and rot:IsA("BasePart") then
            local c0 = rot.CFrame
            local tw = TweenService:Create(rot, TweenInfo.new(3.2, Enum.EasingStyle.Quart, Enum.EasingDirection.Out),
                { CFrame = c0 * CFrame.Angles(0, math.rad(1800 + math.random(0, 360)), 0) })
            tw:Play()
        end
    end)
    task.delay(4.2, function()
        local nm = liveLabel(seg) or (pick and pick.name)
        notify("You unlocked: " .. (nm or "Mystery Prize"), true)
        if fakeHourly > 0 then
            task.delay(1.0, function() playHourlySpin() end)
        end
    end)
    return true
end

local function findHaloContainer()
    local c
    pcall(function()
        local hs = PlayerGui:FindFirstChild("HalloweenSpin")
        local mm = hs and hs:WaitForChild("Main", 3)
        c = mm and mm:FindFirstChild("Container")
    end)
    return c
end

local function playHaloSpin(containerGui)
    local Container = containerGui or findHaloContainer()
    if not Container then return false end
    if fakeHSpins <= 0 then return false end
    fakeHSpins = fakeHSpins - 1
    suppressUntil = os.clock() + 2
    local pick
    pcall(function()
        if HaloMod and HaloMod.Rewards then
            local entries = {}
            local odds
            if HaloMod.GetOdds then
                local ok, o = pcall(HaloMod.GetOdds)
                if ok and type(o) == "table" then odds = o end
            end
            for id, it in pairs(HaloMod.Rewards) do
                if type(it) == "table" and it.Name then
                    local ch = tonumber(it.Chance) or 0
                    if odds and tonumber(odds[id]) then ch = tonumber(odds[id]) end
                    entries[#entries+1] = { key = tostring(id), name = it.Name, chance = ch }
                end
            end
            pick = rollEntries(entries)
        end
    end)
    task.spawn(function()
        local rewardsRoot = Container:FindFirstChild("Rewards")
        local frames, target = {}, nil
        if rewardsRoot then
            for _, ch in ipairs(rewardsRoot:GetChildren()) do
                if ch:IsA("GuiObject") and ch.Visible then
                    frames[#frames+1] = ch
                    if pick and ch.Name == pick.key then target = ch end
                end
            end
        end
        local landed = highlightRun(frames, target)
        local nm = liveLabel(landed) or (pick and pick.name) or pickRewardFromUI(Container)
        notify("You spinned: " .. (nm or "Halloween Prize"), true)
    end)
    return true
end

local function findEmoteMain()
    local m
    pcall(function()
        local eg = PlayerGui:FindFirstChild("EmotesGacha")
        m = eg and eg:WaitForChild("Main", 3)
    end)
    return m
end

local function playEmoteSpin(mainGui, n)
    local Main = mainGui or findEmoteMain()
    if not Main then return false end
    n = tonumber(n) or 1
    suppressUntil = os.clock() + 1.5
    playLocal(spinSound())
    task.spawn(function()
        local frames = {}
        pcall(function()
            local rwd = Main:FindFirstChild("Rewards")
            if rwd then
                for _, ch in ipairs(rwd:GetChildren()) do
                    if ch:IsA("GuiObject") then frames[#frames+1] = ch end
                end
            end
        end)
        for _ = 1, n do
            local pick = rollEntries(flatNested(EmoteRewards))
            local target
            if pick then
                for _, f in ipairs(frames) do if f.Name == pick.key then target = f; break end end
                if not target then for _, f in ipairs(frames) do if liveLabel(f) == pick.name then target = f; break end end end
            end
            highlightRun(frames, target)
            local nm = liveLabel(target) or (pick and pick.name) or pickRewardFromUI(Main)
            notify("You spun: " .. (nm or "Emote"), true)
            if n > 1 then task.wait(0.5) end
        end
    end)
    return true
end

local function seasonTab(which)
    local sp = PlayerGui:FindFirstChild("SeasonPass")
    if not sp then return nil end
    local tab
    pcall(function()
        local m = sp:WaitForChild("Main", 3)
        m = m and m:WaitForChild("Tabs", 3)
        tab = m and m:WaitForChild(which, 3)
    end)
    return tab
end

local function playGachaSpin(tabGui, n)
    local tab = tabGui or seasonTab("Spin")
    if not tab then return false end
    if fakeGacha <= 0 then return false end
    n = tonumber(n) or 1
    fakeGacha = fakeGacha - n; if fakeGacha < 0 then fakeGacha = 0 end
    suppressUntil = os.clock() + 1.5
    playLocal(spinSound())
    task.spawn(function()
        local frames = {}
        pcall(function()
            local rw = tab:FindFirstChild("Rewards")
            if rw then
                for _, ch in ipairs(rw:GetChildren()) do
                    if ch:IsA("GuiObject") then frames[#frames+1] = ch end
                end
            end
        end)
        for _ = 1, n do
            local pick = rollEntries(flatDict(GachaRewards))
            local target
            if pick then
                for _, f in ipairs(frames) do if f.Name == pick.key then target = f; break end end
                if not target then for _, f in ipairs(frames) do if liveLabel(f) == pick.name then target = f; break end end end
            end
            local land = highlightRun(frames, target)
            local nm = liveLabel(land) or (pick and pick.name) or pickRewardFromUI(tab)
            pcall(function()
                local rt = tab:FindFirstChild("RewardText")
                if rt and rt:IsA("TextLabel") then rt.Text = nm or "Spin for REWARD" end
            end)
            notify("You spun: " .. (nm or "Spin Prize"), true)
            if n > 1 then task.wait(0.8) end
        end
    end)
    return true
end

local function playRaffleSpin(tabGui, n)
    local tab = tabGui or seasonTab("Gacha")
    if not tab then return false end
    if fakeRaffle <= 0 then return false end
    n = tonumber(n) or 1
    fakeRaffle = fakeRaffle - n; if fakeRaffle < 0 then fakeRaffle = 0 end
    suppressUntil = os.clock() + 1.5
    playLocal(spinSound())
    task.spawn(function()
        local frames = {}
        local basic = tab:FindFirstChild("Basic")
        local leg = tab:FindFirstChild("Legendary")
        pcall(function()
            if basic then for _, ch in ipairs(basic:GetChildren()) do if ch:IsA("GuiObject") then frames[#frames+1] = ch end end end
            if leg then for _, ch in ipairs(leg:GetChildren()) do if ch:IsA("GuiObject") then frames[#frames+1] = ch end end end
        end)
        for _ = 1, n do
            local pick = rollEntries(flatNested(RaffleRewards))
            local target
            if pick then
                for _, f in ipairs(frames) do if f.Name == pick.key then target = f; break end end
                if not target then for _, f in ipairs(frames) do if liveLabel(f) == pick.name then target = f; break end end end
            end
            local land = highlightRun(frames, target)
            local nm = liveLabel(land) or (pick and pick.name) or pickRewardFromUI(tab) or "Gacha Prize"
            notify("You spun: " .. nm, true)
            if n > 1 then task.wait(0.8) end
        end
    end)
    return true
end

-- ============================================================
-- LUCK WHEEL
-- ============================================================
local function openLuck()
    local opened = false
    pcall(function()
        local gc = require(LocalPlayer:WaitForChild("PlayerScripts"):WaitForChild("Controllers"):WaitForChild("GuiController"))
        if gc then
            if gc.IsOpen and gc.Open and gc:IsOpen("Wheel") then
                gc:Open("LuckWheel")
            elseif gc.ToggleGui then
                gc:ToggleGui("LuckWheel")
            elseif gc.Open then
                gc:Open("LuckWheel")
            end
            opened = true
        end
    end)
    if not opened then
        pcall(function()
            local lw = PlayerGui:FindFirstChild("LuckWheel")
            if lw then lw.Enabled = true end
        end)
    end
    task.spawn(function()
        playLocal(rewardSound())
        notify("Luck Boosted !", true)
        pcall(function()
            local lw = PlayerGui:FindFirstChild("LuckWheel")
            if not lw then return end
            local frame = lw:FindFirstChild("CloverRainFrame")
            local mid = lw:FindFirstChild("Main") and lw.Main:FindFirstChild("Middle")
            local timerLb = mid and mid:FindFirstChild("Timer")
            if frame then
                pcall(function() frame.Transparency = 0.85 end)
                for i = 1, 20 do
                    pcall(function()
                        local c = Instance.new("Frame")
                        c.Size = UDim2.new(0, 42, 0, 21)
                        c.BackgroundTransparency = 1
                        c.Position = UDim2.new(math.random(), 0, 0, -10)
                        c.Rotation = math.random(-10, 10)
                        c.Parent = frame
                        local img = Instance.new("ImageLabel")
                        img.Size = UDim2.new(1, 0, 1, 0)
                        img.BackgroundTransparency = 1
                        img.Image = "rbxassetid://118572781839100"
                        img.Parent = c
                        local tw = TweenService:Create(c, TweenInfo.new(2, Enum.EasingStyle.Linear), { Position = UDim2.new(c.Position.X.Scale, 0, 1, 20) })
                        tw:Play()
                        Debris:AddItem(c, 2.2)
                    end)
                    playTick()
                    task.wait(0.2)
                end
                task.delay(5, function() pcall(function() frame.Transparency = 1 end) end)
            end
            if timerLb and timerLb:IsA("TextLabel") then
                task.spawn(function()
                    for t = 300, 1, -1 do
                        local m = math.floor(t / 60)
                        local s = t % 60
                        local ok = pcall(function() timerLb.Text = "Time Left : " .. string.format("%d:%02d", m, s) end)
                        if not ok then break end
                        task.wait(1)
                    end
                end)
            end
        end)
    end)
end

-- ============================================================
-- WHEEL UI HOOKS
-- ============================================================
local hookedOther = {}
local hourlyWired, hourlyWiredBtn = {}, {}
local hookState = {}

local function realHourlyFree()
    local n = 0
    pcall(function() if DataRef then n = tonumber(DataRef:Get("NewHourlySpin.FreeSpins")) or 0 end end)
    return n
end
local function realTickets()
    local n = 0
    pcall(function() if DataRef then n = tonumber(DataRef:Get("GoldenTickets")) or 0 end end)
    return n
end
local function realSpinCount()
    local n = 0
    pcall(function()
        if DataRef then
            n = (tonumber(DataRef:Get({"SpinWheel","Spins"})) or 0) + (tonumber(DataRef:Get({"SpinWheel","FreeSpins"})) or 0)
        end
    end)
    return n
end

local function paintHourlyHud()
    pcall(function()
        local hw = PlayerGui:FindFirstChild("HourlyWheel")
        local m = hw and hw:FindFirstChild("Main")
        if not m then return end
        local sb = m:FindFirstChild("HourlySpinBtn"); if sb then sb.Visible = false end
        local sp = m:FindFirstChild("Spins")
        if sp and sp:IsA("TextLabel") and fakeHourly > 0 then
            sp.Text = "Spins: " .. tostring(fakeHourly); sp.Visible = true
        end
    end)
end

task.spawn(function()
    while true do pcall(paintHourlyHud); task.wait(0.5) end
end)

local function hookHourly()
    local hw = PlayerGui:FindFirstChild("HourlyWheel")
    if not hw then return end
    local Main = hw:FindFirstChild("Main")
    if not Main then pcall(function() Main = hw:WaitForChild("Main", 3) end) end
    if not Main then return end
    hookState.H = true
    pcall(paintHourlyHud)
    hookedOther["hourly"] = true
    local btn = Main:FindFirstChild("HourlySpinBtn")
    local buy3 = Main:FindFirstChild("Buy3Spin")
    local buy10 = Main:FindFirstChild("Buy10Spin")
    local prods = Main:FindFirstChild("Products")
    local prod3 = prods and prods:FindFirstChild("Spin3")
    local prod10 = prods and prods:FindFirstChild("Spin10")
    local function wire(key, b, fn)
        if not b or hourlyWired[key] then return end
        disconnectGame(b)
        local ok = pcall(function() b.MouseButton1Click:Connect(fn) end)
        if ok then hourlyWired[key] = true; hourlyWiredBtn[b] = true end
    end
    local function grantHourly(n)
        fakeHourly = fakeHourly + n
        suppressUntil = os.clock() + 1
        notify("+" .. tostring(n) .. " Wheel Spins", true)
        pcall(paintHourlyHud)
    end
    local function hourlyBuyFlow(count, btn)
        local pid = count >= 10 and PID.hourly10 or PID.hourly3
        local cost = (btn and priceOf(btn)) or (count >= 10 and H_BUY10 or (count >= 3 and H_BUY3 or H_SPIN))
        if pid == 0 then
            if net < cost then notify("Need " .. fmtRaw(cost) .. " Tokens", false); return end
            net = net - cost; pcall(spoofHUD); grantHourly(count)
            task.delay(0.6, function() playHourlySpin(Main) end)
            return
        end
        openChoice(pid, cost, function()
            grantHourly(count)
            task.delay(0.6, function() playHourlySpin(Main) end)
        end)
    end
    wire("hourly-prod3", prod3, function() hourlyBuyFlow(3, prod3) end)
    wire("hourly-prod10", prod10, function() hourlyBuyFlow(10, prod10) end)
    wire("hourly-buy3", buy3, function() hourlyBuyFlow(3, buy3) end)
    wire("hourly-buy10", buy10, function() hourlyBuyFlow(10, buy10) end)
    wire("hourly-spin", btn, function()
        if fakeHourly > 0 then playHourlySpin(Main); return end
        if PID.hourly3 == 0 then notify("Buy spins first", false); return end
        local pb3 = Main:FindFirstChild("Buy3Spin")
        openChoice(PID.hourly3, (priceOf(pb3) or H_BUY3), function()
            fakeHourly = fakeHourly + 3
            notify("+3 Wheel Spins", true)
            task.delay(0.6, function() playHourlySpin(Main) end)
        end)
    end)
end

local function hookFestival()
    local fw = PlayerGui:FindFirstChild("FestivalWheel")
    if not fw then return end
    local ok, Main = pcall(function() return fw:WaitForChild("Main", 3) end)
    if not ok or not Main then return end
    if hookedOther["festival"] then hookState.F = true; return end
    hookedOther["festival"] = true; hookState.F = true
    local btn = Main:WaitForChild("SpinButton", 5)
    disconnectGame(btn)
    pcall(function()
        if btn then
            btn.MouseButton1Click:Connect(function()
                if fakeTickets <= 0 then
                    if net >= F_TICKET then
                        net = net - F_TICKET; fakeTickets = fakeTickets + 1
                        pcall(spoofHUD); notify("-" .. fmtRaw(F_TICKET) .. " Tokens", true)
                    else notify("You don't have enough Golden Tickets!", false); return end
                end
                fakeTickets = fakeTickets - 1
                suppressUntil = os.clock() + 1.5
                local entries = {}
                pcall(function()
                    if FestCrates and FestCrates.Basic then
                        for i, it in pairs(FestCrates.Basic) do
                            if type(it) == "table" and it.Name then
                                local kk = it.Name
                                if kk == "" then kk = tostring(i) end
                                entries[#entries+1] = { key = kk, name = it.Name, chance = tonumber(it.Chance) or 0 }
                            end
                        end
                    end
                end)
                local pick = rollEntries(entries)
                local spinner = Main:FindFirstChild("Spinner")
                local seg
                if spinner then
                    if pick then
                        pcall(function()
                            seg = spinner:FindFirstChild(pick.key)
                            if not seg and tonumber(pick.key) then seg = spinner:FindFirstChild(tostring(tonumber(pick.key))) end
                        end)
                    end
                    tweenToSeg(spinner, seg, 4)
                else playLocal(spinSound()) end
                task.delay(4.2, function()
                    local nm = liveLabel(seg) or (pick and pick.name)
                    if not nm then nm = pickRewardFromUI(Main) end
                    notify("You unlocked: " .. (nm or "Festival Prize"), true)
                end)
            end)
        end
    end)
end

local function hookHalloween()
    local hs = PlayerGui:FindFirstChild("HalloweenSpin")
    if not hs then return end
    local ok, Main = pcall(function() return hs:WaitForChild("Main", 3) end)
    if not ok or not Main then return end
    local Container = Main:FindFirstChild("Container")
    if not Container then return end
    if hookedOther["halloween"] then hookState.Hl = true; return end
    hookedOther["halloween"] = true; hookState.Hl = true
    local spin = Container:FindFirstChild("Spin")
    local buy1 = Container:FindFirstChild("Buy1")
    local buy10 = Container:FindFirstChild("Buy10")
    disconnectGame(spin); disconnectGame(buy1); disconnectGame(buy10)
    pcall(function()
        if buy1 then
            buy1.Activated:Connect(function()
                if PID.halo1 == 0 then
                    if net < H_SPIN_COST then notify("Need " .. fmtRaw(H_SPIN_COST) .. " Tokens", false); return end
                    net = net - H_SPIN_COST; fakeHSpins = fakeHSpins + 1
                    suppressUntil = os.clock() + 1
                    pcall(spoofHUD); notify("+1 Halloween Spin", true); return
                end
                openChoice(PID.halo1, (priceOf(buy1) or H_SPIN_COST), function()
                    fakeHSpins = fakeHSpins + 1
                    suppressUntil = os.clock() + 1
                    notify("+1 Halloween Spin", true)
                end)
            end)
        end
        if buy10 then
            buy10.Activated:Connect(function()
                if PID.halo10 == 0 then
                    if net < H_BUY10_COST then notify("Need " .. fmtRaw(H_BUY10_COST) .. " Tokens", false); return end
                    net = net - H_BUY10_COST; fakeHSpins = fakeHSpins + 10
                    suppressUntil = os.clock() + 1
                    pcall(spoofHUD); notify("+10 Halloween Spins", true); return
                end
                openChoice(PID.halo10, (priceOf(buy10) or H_BUY10_COST), function()
                    fakeHSpins = fakeHSpins + 10
                    suppressUntil = os.clock() + 1
                    notify("+10 Halloween Spins", true)
                end)
            end)
        end
        if spin then
            spin.Activated:Connect(function()
                if fakeHSpins > 0 then playHaloSpin(Container); return end
                if PID.halo1 == 0 then notify("Not enough bones!", false); return end
                openChoice(PID.halo1, (priceOf(buy1) or H_SPIN_COST), function()
                    fakeHSpins = fakeHSpins + 1
                    notify("+1 Halloween Spin", true)
                    task.delay(0.6, function() playHaloSpin(Container) end)
                end)
            end)
        end
    end)
end

local function hookEmotes()
    local eg = PlayerGui:FindFirstChild("EmotesGacha")
    if not eg then return end
    local ok, Main = pcall(function() return eg:WaitForChild("Main", 3) end)
    if not ok or not Main then return end
    if hookedOther["emotes"] then hookState.E = true; return end
    hookedOther["emotes"] = true; hookState.E = true
    local s1 = Main:FindFirstChild("Spin1")
    local s10 = Main:FindFirstChild("Spin10")
    disconnectGame(s1); disconnectGame(s10)
    local function hookOne(btn, fn)
        if not btn then return end
        pcall(function()
            if btn.Activated then btn.Activated:Connect(fn)
            else btn.MouseButton1Click:Connect(fn) end
        end)
    end
    pcall(function()
        hookOne(s1, function()
            if PID.emote1 == 0 then
                if net < E_SPIN1 then notify("Need " .. fmtRaw(E_SPIN1) .. " Tokens", false); return end
                net = net - E_SPIN1; pcall(spoofHUD)
                notify("+1 Emote Spin", true); playEmoteSpin(Main, 1); return
            end
            openChoice(PID.emote1, (priceOf(s1) or E_SPIN1), function()
                notify("+1 Emote Spin", true)
                task.delay(0.6, function() playEmoteSpin(Main, 1) end)
            end)
        end)
        hookOne(s10, function()
            if PID.emote10 == 0 then
                if net < E_SPIN10 then notify("Need " .. fmtRaw(E_SPIN10) .. " Tokens", false); return end
                net = net - E_SPIN10; pcall(spoofHUD)
                notify("+10 Emote Spins", true); playEmoteSpin(Main, 10); return
            end
            openChoice(PID.emote10, (priceOf(s10) or E_SPIN10), function()
                notify("+10 Emote Spins", true)
                task.delay(0.6, function() playEmoteSpin(Main, 10) end)
            end)
        end)
    end)
end

local function hookSeasonSpin()
    local tab = seasonTab("Spin")
    if not tab then return end
    if hookedOther["sspin"] then hookState.SS = true; return end
    hookedOther["sspin"] = true; hookState.SS = true
    local spinB = tab:FindFirstChild("Spin")
    local s1 = tab:FindFirstChild("Spin1")
    local s10 = tab:FindFirstChild("Spin10")
    disconnectGame(spinB); disconnectGame(s1); disconnectGame(s10)
    pcall(function()
        if s1 then
            s1.MouseButton1Click:Connect(function()
                if not canBuyGate("Gacha_CanBuy") then return end
                if PID.gacha1 == 0 then
                    if net < G_SPIN1 then notify("Need " .. fmtRaw(G_SPIN1) .. " Tokens", false); return end
                    net = net - G_SPIN1; fakeGacha = fakeGacha + 1
                    pcall(spoofHUD); notify("+1 Spin", true); return
                end
                openChoice(PID.gacha1, (priceOf(s1) or G_SPIN1), function()
                    fakeGacha = fakeGacha + 1; notify("+1 Spin", true)
                end)
            end)
        end
        if s10 then
            s10.MouseButton1Click:Connect(function()
                if not canBuyGate("Gacha_CanBuy") then return end
                if PID.gacha10 == 0 then
                    if net < G_SPIN10 then notify("Need " .. fmtRaw(G_SPIN10) .. " Tokens", false); return end
                    net = net - G_SPIN10; fakeGacha = fakeGacha + 10
                    pcall(spoofHUD); notify("+10 Spins", true); return
                end
                openChoice(PID.gacha10, (priceOf(s10) or G_SPIN10), function()
                    fakeGacha = fakeGacha + 10; notify("+10 Spins", true)
                end)
            end)
        end
        if spinB then
            spinB.MouseButton1Click:Connect(function()
                if fakeGacha > 0 then playGachaSpin(tab, 1); return end
                if PID.gacha1 == 0 then notify("Buy spins first", false); return end
                openChoice(PID.gacha1, (priceOf(s1) or G_SPIN1), function()
                    fakeGacha = fakeGacha + 1
                    notify("+1 Spin", true)
                    task.delay(0.6, function() playGachaSpin(tab, 1) end)
                end)
            end)
        end
    end)
end

local function hookSeasonGacha()
    local tab = seasonTab("Gacha")
    if not tab then return end
    if hookedOther["sgacha"] then hookState.SG = true; return end
    hookedOther["sgacha"] = true; hookState.SG = true
    local spinB = tab:FindFirstChild("Spin")
    local s1 = tab:FindFirstChild("Spin1")
    local s10 = tab:FindFirstChild("Spin10")
    disconnectGame(spinB); disconnectGame(s1); disconnectGame(s10)
    pcall(function()
        if s1 then
            s1.MouseButton1Click:Connect(function()
                if not canBuyGate("RaffleGacha_CanBuy") then return end
                if PID.raffle1 == 0 then
                    if net < R_SPIN1 then notify("Need " .. fmtRaw(R_SPIN1) .. " Tokens", false); return end
                    net = net - R_SPIN1; fakeRaffle = fakeRaffle + 1
                    pcall(spoofHUD); notify("+1 Spin", true); return
                end
                openChoice(PID.raffle1, (priceOf(s1) or R_SPIN1), function()
                    fakeRaffle = fakeRaffle + 1; notify("+1 Spin", true)
                end)
            end)
        end
        if s10 then
            s10.MouseButton1Click:Connect(function()
                if not canBuyGate("RaffleGacha_CanBuy") then return end
                if PID.raffle10 == 0 then
                    if net < R_SPIN10 then notify("Need " .. fmtRaw(R_SPIN10) .. " Tokens", false); return end
                    net = net - R_SPIN10; fakeRaffle = fakeRaffle + 10
                    pcall(spoofHUD); notify("+10 Spins", true); return
                end
                openChoice(PID.raffle10, (priceOf(s10) or R_SPIN10), function()
                    fakeRaffle = fakeRaffle + 10; notify("+10 Spins", true)
                end)
            end)
        end
        if spinB then
            spinB.MouseButton1Click:Connect(function()
                if fakeRaffle > 0 then playRaffleSpin(tab, 1); return end
                if PID.raffle1 == 0 then notify("Buy spins first", false); return end
                openChoice(PID.raffle1, (priceOf(s1) or R_SPIN1), function()
                    fakeRaffle = fakeRaffle + 1
                    notify("+1 Spin", true)
                    task.delay(0.6, function() playRaffleSpin(tab, 1) end)
                end)
            end)
        end
    end)
end

local genericDone = {}
local function hookGenericSpins()
    for _, g in ipairs(PlayerGui:GetChildren()) do
        if g:IsA("ScreenGui") and g.Name ~= "TokenPanel" then
            local nm = g.Name:lower()
            if nm:find("wheel") or nm:find("spin") or nm:find("gacha") or nm:find("crate") or nm:find("emotes") then
                local hookedName = (g.Name == "Wheel" or g.Name == "HourlyWheel" or g.Name == "FestivalWheel"
                    or g.Name == "HalloweenSpin" or g.Name == "EmotesGacha")
                if not hookedName then
                    for _, d in ipairs(g:GetDescendants()) do
                        if d:IsA("GuiButton") and (d.Name == "Spin" or d.Name == "SpinButton") then
                            local key = "gen:" .. d:GetFullName()
                            if not genericDone[key] then
                                genericDone[key] = true
                                local function genSpin()
                                    if net < SPIN_COST then return end
                                    net = net - SPIN_COST
                                    suppressUntil = os.clock() + 2
                                    pcall(spoofHUD); playLocal(spinSound())
                                    task.delay(2.5, function()
                                        local rnm = pickRewardFromUI(g)
                                        notify("You unlocked: " .. (rnm or "Mystery Prize"), true)
                                    end)
                                end
                                pcall(function()
                                    if d.Activated then d.Activated:Connect(genSpin)
                                    else d.MouseButton1Click:Connect(genSpin) end
                                end)
                            end
                        end
                    end
                end
            end
        end
    end
end

local function findSpinUI()
    local wheel = PlayerGui:FindFirstChild("Wheel")
    if not wheel then return nil end
    local container
    pcall(function()
        local m = wheel:FindFirstChild("Main", true)
        if m then container = m.Parent end
    end)
    if not container then return nil end
    local ok, res = pcall(function()
        local Main = container:WaitForChild("Main", 3)
        local Grid = container:FindFirstChild("Grid")
        if not Main then return nil end
        return {
            Main = Main, Grid = Grid,
            Spinner = Main:FindFirstChild("Spinner"),
            Spins = Main:FindFirstChild("Spins"),
            SpinButton = Main:FindFirstChild("SpinButton"),
            Buy1Spin = Main:FindFirstChild("Buy1Spin"),
            Extra3 = Main:FindFirstChild("ExtraButtons") and Main.ExtraButtons:FindFirstChild("3Spins"),
            Extra10 = Main:FindFirstChild("ExtraButtons") and Main.ExtraButtons:FindFirstChild("10Spins"),
            GridSpinButton = Grid and Grid:FindFirstChild("SpinButton"),
            GridSpins = Grid and Grid:FindFirstChild("Spins"),
            GridBuy1 = Grid and Grid:FindFirstChild("Buy1Spin"),
            GridBuy3 = Grid and Grid:FindFirstChild("Buy3Spin"),
            GridBuy10 = Grid and Grid:FindFirstChild("Buy10Spin"),
            Prod1 = Main:FindFirstChild("Products") and Main.Products:FindFirstChild("Spin1"),
            Prod3 = Main:FindFirstChild("Products") and Main.Products:FindFirstChild("Spin3"),
            Prod10 = Main:FindFirstChild("Products") and Main.Products:FindFirstChild("Spin10"),
        }
    end)
    if ok then return res end
end

local WHEEL_REWARDS = {
    "Spiked Purse","Yellow Bear Purse","Loveboard","Epic Graffiti","Legend Hoverboard","Spiked Knife",
}

local function paintSpins(ui)
    if not ui then return end
    local total = realSpinCount() + fakeSpins
    pcall(function()
        if ui.Spins and ui.Spins:IsA("TextLabel") then
            ui.Spins.Text = string.format("Total Spins: %s", tostring(total)); ui.Spins.Visible = true
        end
        if ui.GridSpins and ui.GridSpins:IsA("TextLabel") then
            ui.GridSpins.Text = string.format("Total Spins: %s", tostring(total)); ui.GridSpins.Visible = true
        end
        if fakeSpins > 0 then
            if ui.SpinButton then ui.SpinButton.Visible = true end
            if ui.GridSpinButton then ui.GridSpinButton.Visible = true end
            if ui.Buy1Spin then ui.Buy1Spin.Visible = false end
            if ui.GridBuy1 then ui.GridBuy1.Visible = false end
        end
    end)
end

local function doWheelSpinNow(ui)
    fakeSpins = fakeSpins - 1
    suppressUntil = os.clock() + 1.5
    paintSpins(ui)
    local entries = {}
    pcall(function()
        if type(WheelOdds) == "table" then
            for name, ch in pairs(WheelOdds) do
                local disp = name
                if WheelNames and WheelNames[name] then disp = WheelNames[name] end
                entries[#entries+1] = { key = tostring(name), name = disp, chance = tonumber(ch) or 0 }
            end
        end
    end)
    local pick = rollEntries(entries)
    local gridOn = false
    pcall(function() if ui.Grid and ui.Grid.Visible then gridOn = true end end)
    if gridOn and ui.Grid then
        local frames = {}
        pcall(function()
            local rl = ui.Grid:FindFirstChild("RewardsList")
            local mm = rl and rl:FindFirstChild("Main")
            local rw = mm and mm:FindFirstChild("Rewards")
            if rw then
                for _, ch in ipairs(rw:GetChildren()) do
                    if ch:IsA("ImageLabel") and ch:GetAttribute("Reward") then frames[#frames+1] = ch end
                end
            end
        end)
        task.spawn(function()
            local target
            if pick then
                for _, f in ipairs(frames) do
                    local ok, rn = pcall(function() return f:GetAttribute("Reward") end)
                    if ok and (rn == pick.name or rn == pick.key) then target = f; break end
                end
            end
            local land = highlightRun(frames, target)
            local nm = liveLabel(land) or (pick and pick.name)
            notify("You unlocked: " .. (nm or WHEEL_REWARDS[math.random(1, #WHEEL_REWARDS)]), true)
            paintSpins(ui)
        end)
        return
    end
    local spinner = ui.Spinner
    local seg
    if spinner and spinner:IsA("GuiObject") then
        if pick then pcall(function() seg = spinner:FindFirstChild(pick.name) end) end
        if not seg then
            pcall(function()
                local kids = {}
                for _, ch in ipairs(spinner:GetChildren()) do if ch:IsA("GuiObject") then kids[#kids+1] = ch end end
                if #kids > 0 then seg = kids[math.random(1, #kids)] end
            end)
        end
        tweenToSeg(spinner, seg, WheelAnimT, WheelAnimStyle, WheelAnimDir)
    else playLocal(spinSound()) end
    task.delay(4.2, function()
        local nm = liveLabel(seg) or (pick and pick.name)
        if not nm then nm = pickRewardFromUI(ui.Main) or pickRewardFromUI(ui.Grid) end
        notify("You unlocked: " .. (nm or WHEEL_REWARDS[math.random(1, #WHEEL_REWARDS)]), true)
        pcall(function()
            local damt = dineroAmount(pick and pick.key, nm)
            if damt and damt > 0 then netDinero = netDinero + damt; spoofHUD() end
        end)
        paintSpins(ui)
    end)
end

local function spin1Cost(ui)
    return (ui and (priceOf(ui.Buy1Spin) or priceOf(ui.GridBuy1))) or SPIN1
end

local function wheelOneProduct()
    local id = PID.wheel1
    pcall(function()
        local sw = ReplicatedStorage:WaitForChild("SpinWheel", 2)
        if sw and sw.IsOfferTime and PID.wheel1Offer ~= 0 and sw.IsOfferTime:InvokeServer() then
            id = PID.wheel1Offer
        end
    end)
    return id
end

local function doWheelSpin(ui)
    if fakeSpins > 0 then doWheelSpinNow(ui); return end
    local pid = wheelOneProduct()
    if pid == 0 then notify("Buy spins first", false); return end
    openChoice(pid, spin1Cost(ui), function()
        fakeSpins = fakeSpins + 1
        paintSpins(ui)
        notify("+1 Wheel Spin", true)
        task.delay(0.6, function() doWheelSpinNow(ui) end)
    end)
end

local function doBuySpins(ui, count, cost, productId)
    local pid = productId or 0
    if pid == 0 then
        if net < cost then notify("Need " .. fmtRaw(cost) .. " Tokens", false); return end
        net = net - cost; fakeSpins = fakeSpins + count
        suppressUntil = os.clock() + 1
        pcall(spoofHUD); paintSpins(ui)
        notify("+" .. tostring(count) .. " Wheel Spin", true); return
    end
    openChoice(pid, cost, function()
        fakeSpins = fakeSpins + count
        suppressUntil = os.clock() + 1
        paintSpins(ui)
        notify("+" .. tostring(count) .. " Wheel Spin", true)
    end)
end

local hookedBtns = {}
local function hookWheelUI(ui)
    if not ui or not ui.Main then return false end
    local key = "wheel"
    pcall(function() key = tostring(ui.Main:GetDebugId()) end)
    if key == "wheel" then pcall(function() key = ui.Main:GetFullName() end) end
    if hookedBtns[key] then paintSpins(ui); hookState.W = true; return true end
    hookedBtns[key] = true
    disconnectGame(ui.SpinButton); disconnectGame(ui.GridSpinButton)
    disconnectGame(ui.Buy1Spin);   disconnectGame(ui.GridBuy1)
    disconnectGame(ui.GridBuy3);   disconnectGame(ui.GridBuy10)
    if ui.Extra3 then disconnectGame(ui.Extra3) end
    if ui.Extra10 then disconnectGame(ui.Extra10) end
    if ui.Prod1 then disconnectGame(ui.Prod1) end
    if ui.Prod3 then disconnectGame(ui.Prod3) end
    if ui.Prod10 then disconnectGame(ui.Prod10) end
    pcall(function()
        if ui.SpinButton then ui.SpinButton.MouseButton1Click:Connect(function() doWheelSpin(ui) end) end
        if ui.GridSpinButton then ui.GridSpinButton.MouseButton1Click:Connect(function() doWheelSpin(ui) end) end
        if ui.Buy1Spin then ui.Buy1Spin.MouseButton1Click:Connect(function() doBuySpins(ui, 1, (priceOf(ui.Buy1Spin) or SPIN1), wheelOneProduct()) end) end
        if ui.GridBuy1 then ui.GridBuy1.MouseButton1Click:Connect(function() doBuySpins(ui, 1, (priceOf(ui.GridBuy1) or SPIN1), wheelOneProduct()) end) end
        if ui.GridBuy3 then ui.GridBuy3.MouseButton1Click:Connect(function() doBuySpins(ui, 3, (priceOf(ui.GridBuy3) or SPIN3), PID.wheel3) end) end
        if ui.GridBuy10 then ui.GridBuy10.MouseButton1Click:Connect(function() doBuySpins(ui, 10, (priceOf(ui.GridBuy10) or SPIN10), PID.wheel10) end) end
        if ui.Extra3 then ui.Extra3.MouseButton1Click:Connect(function() doBuySpins(ui, 3, (priceOf(ui.Extra3) or SPIN3), PID.wheel3) end) end
        if ui.Extra10 then ui.Extra10.MouseButton1Click:Connect(function() doBuySpins(ui, 10, (priceOf(ui.Extra10) or SPIN10), PID.wheel10) end) end
        if ui.Prod1 then ui.Prod1.MouseButton1Click:Connect(function() doBuySpins(ui, 1, (priceOf(ui.Prod1) or SPIN1), wheelOneProduct()) end) end
        if ui.Prod3 then ui.Prod3.MouseButton1Click:Connect(function() doBuySpins(ui, 3, (priceOf(ui.Prod3) or SPIN3), PID.wheel3) end) end
        if ui.Prod10 then ui.Prod10.MouseButton1Click:Connect(function() doBuySpins(ui, 10, (priceOf(ui.Prod10) or SPIN10), PID.wheel10) end) end
    end)
    paintSpins(ui)
    hookState.W = true
    return true
end

-- ============================================================
-- MARKET REROUTE
-- ============================================================
local productSpin = {}
local function rebuildProductMap()
    local m = {}
    local function reg(pid, kind, count)
        if type(pid) == "number" and pid ~= 0 then m[pid] = { kind = kind, count = count } end
    end
    reg(PID.wheel1, "wheel", 1); reg(PID.wheel1Offer, "wheel", 1)
    reg(PID.wheel3, "wheel", 3); reg(PID.wheel10, "wheel", 10)
    reg(PID.hourly3, "hourly", 3); reg(PID.hourly10, "hourly", 10)
    reg(PID.halo1, "halo", 1); reg(PID.halo10, "halo", 10)
    reg(PID.emote1, "emote", 1); reg(PID.emote10, "emote", 10)
    reg(PID.gacha1, "gacha", 1); reg(PID.gacha10, "gacha", 10)
    reg(PID.raffle1, "raffle", 1); reg(PID.raffle10, "raffle", 10)
    productSpin = m
end

local function grantAndPlay(kind, count)
    pcall(spoofHUD)
    count = tonumber(count) or 1
    if kind == "wheel" then
        fakeSpins = fakeSpins + count
        notify("+" .. tostring(count) .. " Wheel Spin", true)
        pcall(function() local ui = findSpinUI(); if ui then doWheelSpinNow(ui) end end)
    elseif kind == "hourly" then
        fakeHourly = fakeHourly + count
        notify("+" .. tostring(count) .. " Wheel Spins", true)
        pcall(function() playHourlySpin() end)
    elseif kind == "halo" then
        fakeHSpins = fakeHSpins + count
        notify("+" .. tostring(count) .. " Halloween Spin", true)
        pcall(function() playHaloSpin() end)
    elseif kind == "emote" then
        notify("+" .. tostring(count) .. " Emote Spin", true)
        pcall(function() playEmoteSpin(nil, count) end)
    elseif kind == "gacha" then
        fakeGacha = fakeGacha + count
        notify("+" .. tostring(count) .. " Spin", true)
        pcall(function() playGachaSpin(nil, 1) end)
    elseif kind == "raffle" then
        fakeRaffle = fakeRaffle + count
        notify("+" .. tostring(count) .. " Spin", true)
        pcall(function() playRaffleSpin(nil, 1) end)
    end
end

pcall(function()
    if Market and Market._promptDeveloperProduct then
        local origDP = Market._promptDeveloperProduct
        Market._promptDeveloperProduct = function(self, player, productId)
            if type(productId) ~= "number" then return origDP(self, player, productId) end
            local ok = pcall(function()
                rebuildProductMap()
                local entry = productSpin[productId]
                if entry then
                    local cost = 2500
                    if entry.count >= 10 then cost = 15000 elseif entry.count >= 3 then cost = 6000 end
                    pcall(function()
                        local c = Catalog[productId]
                        if c and type(c.tok) == "number" then cost = c.tok end
                    end)
                    openChoice(productId, cost, function() grantAndPlay(entry.kind, entry.count) end, "Product")
                else
                    openChoice(productId, 0, function() notify("Purchased", true) end, "Product", "soon")
                end
            end)
            if not ok then return origDP(self, player, productId) end
        end
    end
    if Market and Market._promptGamepass then
        local origGP = Market._promptGamepass
        Market._promptGamepass = function(self, player, passId)
            if type(passId) ~= "number" then return origGP(self, player, passId) end
            local ok = pcall(function()
                openChoice(passId, 0, function() notify("Purchased", true) end, "GamePass", "soon")
            end)
            if not ok then return origGP(self, player, passId) end
        end
    end
end)

-- ============================================================
-- TIMED CLAIM CORE
-- ============================================================
local function grant(kind, amount)
    if kind == "token" then
        net = net + amount
        notify("+" .. fmtCommas(amount) .. " Tokens", true)
        if Rayfield then Rayfield:Notify({ Title = "Tokens", Content = "+" .. fmtCommas(amount) .. " Tokens", Duration = 3 }) end
    elseif kind == "dinero" then
        netDinero = netDinero + amount
        notify("+" .. fmtCommas(amount) .. " Dinero", true)
        if Rayfield then Rayfield:Notify({ Title = "Dinero", Content = "+" .. fmtCommas(amount) .. " Dinero", Duration = 3 }) end
    elseif kind == "slay" then
        netSlays = netSlays + amount
        notify("+" .. fmtCommas(amount) .. " Slays", true)
        if Rayfield then Rayfield:Notify({ Title = "Slays", Content = "+" .. fmtCommas(amount) .. " Slays", Duration = 3 }) end
    end
    pcall(spoofHUD)
end

local function setStatus(kind, text)
    local lbl = statusLabels[kind]
    if lbl then pcall(function() lbl:Set(text) end) end
end

local function startTimedClaim(kind, amount, waitSeconds, labelText)
    if activeJob[kind] then notify(kind:upper() .. " job already running", false); return end
    activeJob[kind] = { amount = amount, endTime = os.clock() + waitSeconds, label = labelText }
    notify("Starting: " .. labelText .. " (" .. fmtTime(waitSeconds) .. ")", false)
    task.spawn(function()
        while activeJob[kind] do
            local remaining = activeJob[kind].endTime - os.clock()
            if remaining <= 0 then
                local amt = activeJob[kind].amount
                activeJob[kind] = nil
                setStatus(kind, "Ready")
                grant(kind, amt)
                break
            end
            setStatus(kind, "Running: " .. fmtTime(remaining) .. "  (+" .. fmtCommas(activeJob[kind].amount) .. ")")
            task.wait(0.25)
        end
    end)
end

local function skipJob(kind)
    if not activeJob[kind] then notify("No " .. kind .. " job running", false); return end
    local amt = activeJob[kind].amount
    activeJob[kind] = nil
    setStatus(kind, "Ready")
    grant(kind, amt)
end

local function claimToken(i)
    local o = TOKEN_OPTIONS[i]; if not o then return end
    startTimedClaim("token", o.Amount, o.Wait, o.Label)
end
local function claimDinero(i)
    local o = DINERO_OPTIONS[i]; if not o then return end
    startTimedClaim("dinero", o.Amount, o.Wait, o.Label)
end
local function claimSlays(i)
    local o = SLAY_OPTIONS[i]; if not o then return end
    startTimedClaim("slay", o.Amount, o.Wait, o.Label)
end

-- ============================================================
-- RAYFIELD UI
-- ============================================================
local ok, result = pcall(function()
    return loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
end)
if not ok or not result then __tpPush("RAYFIELD", tostring(result)); error("Rayfield load failed") end
Rayfield = result

local Window = Rayfield:CreateWindow({
    Name = "Baddies GUI â¢ Freeze Trade",
    LoadingTitle = "Baddies GUI",
    LoadingSubtitle = "by Legitness",
    ConfigurationSaving = { Enabled = false },
    Discord = { Enabled = false },
    KeySystem = false,
})

-- TAB: Givers
local GiverTab = Window:CreateTab("Givers", 4483362458)
GiverTab:CreateSection("Tokens")
statusLabels.token = GiverTab:CreateLabel("Ready")
for i, o in ipairs(TOKEN_OPTIONS) do
    GiverTab:CreateButton({ Name = o.Label, Callback = function() claimToken(i) end })
end
GiverTab:CreateSection("Dinero")
statusLabels.dinero = GiverTab:CreateLabel("Ready")
for i, o in ipairs(DINERO_OPTIONS) do
    GiverTab:CreateButton({ Name = o.Label, Callback = function() claimDinero(i) end })
end
GiverTab:CreateSection("Slays")
statusLabels.slay = GiverTab:CreateLabel("Ready")
for i, o in ipairs(SLAY_OPTIONS) do
    GiverTab:CreateButton({ Name = o.Label, Callback = function() claimSlays(i) end })
end

-- TAB: Weapons
local WeaponTab = Window:CreateTab("Weapons", 4483362458)

local filteredCount = 0
for _ in pairs(FilteredWeapons) do filteredCount = filteredCount + 1 end

WeaponTab:CreateSection("Give Weapon (" .. filteredCount .. " tradeable)")
local WeaponInput = WeaponTab:CreateInput({
    Name = "Weapon Name",
    PlaceholderText = "Type weapon name...",
    RemoveTextAfterFocusLost = false,
    Callback = function(txt) end,
})

WeaponTab:CreateButton({
    Name = "Give Weapon",
    Callback = function()
        local name = WeaponInput.Text
        if not name or name == "" then
            Rayfield:Notify({ Title = "Weapons", Content = "Enter a weapon name", Duration = 2 })
            return
        end
        local success = AdminGiveWeapon(name)
        Rayfield:Notify({
            Title = "Weapons",
            Content = success and ("Gave: " .. name) or ("Failed: " .. name),
            Duration = 2,
        })
    end,
})

WeaponTab:CreateSection("Bulk")
WeaponTab:CreateButton({
    Name = "Give All Weapons",
    Callback = function()
        local given, skipped = 0, 0
        for weaponName in pairs(FilteredWeapons) do
            local ok = AdminGiveWeapon(weaponName, true)
            if ok then given = given + 1 else skipped = skipped + 1 end
            task.wait(0.05)
        end
        Rayfield:Notify({
            Title = "Give All",
            Content = "Gave " .. given .. " â¢ skipped " .. skipped,
            Duration = 3,
        })
        notify("Gave " .. given .. " (skipped " .. skipped .. ")", true)
    end,
})

WeaponTab:CreateButton({
    Name = "Duplicate All Items",
    Callback = function() AdminDuplicateItems() end,
})

-- TAB: Wheels
local WheelTab = Window:CreateTab("Wheels", 4483362458)

local function wheelModels()
    local out, seen = {}, {}
    local function push(m) if m and not seen[m] then seen[m] = true; table.insert(out, m) end end
    pcall(function()
        local a = Workspace:FindFirstChild("SpinWheelModel"); if a then push(a) end
        local b = Workspace:FindFirstChild("FestiveSpinWheel"); if b then push(b) end
        local hw = Workspace:FindFirstChild("HalloweenWheel"); if hw then push(hw) end
        for _, c in ipairs(Workspace:GetChildren()) do
            if (c:IsA("Model") or c:IsA("Folder")) and c.Name:lower():find("wheel") then
                if c:FindFirstChildWhichIsA("BasePart", true) then push(c) end
            end
        end
    end)
    return out
end

local function anchorOf(m)
    local ok, p = pcall(function()
        if m:FindFirstChild("GrandPrizePart") and m.GrandPrizePart:IsA("BasePart") then return m.GrandPrizePart end
        if m:FindFirstChild("RotatingPart") and m.RotatingPart:IsA("BasePart") then return m.RotatingPart end
        if m:IsA("Model") and m.PrimaryPart then return m.PrimaryPart end
        return m:FindFirstChildWhichIsA("BasePart", true)
    end)
    if ok then return p end
end

local function tpToWheel(name)
    local m = Workspace:FindFirstChild(name)
    if not m then
        for _, w in ipairs(wheelModels()) do
            if w.Name == name then m = w; break end
        end
    end
    if not m then notify("Wheel not found", false); return end
    local a = anchorOf(m); if not a then return end
    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart", 5)
    if not hrp then return end
    pcall(function()
        TweenService:Create(hrp, TweenInfo.new(1.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            { CFrame = a.CFrame + Vector3.new(0, 3, 6) }):Play()
    end)
end

WheelTab:CreateSection("Travel")
WheelTab:CreateButton({ Name = "TP Hourly Wheel",    Callback = function() tpToWheel("SpinWheelModel") end })
WheelTab:CreateButton({ Name = "TP Festival Wheel",  Callback = function() tpToWheel("FestiveSpinWheel") end })
WheelTab:CreateButton({ Name = "TP Halloween Wheel", Callback = function() tpToWheel("HalloweenWheel") end })

WheelTab:CreateSection("Luck")
WheelTab:CreateButton({
    Name = "Luck Wheel (300s boost)",
    Callback = function() openLuck() end,
})

-- TAB: Dupe
local DupeTab = Window:CreateTab("Dupe [NEW]", 4483362458)

local function handleDupeButton(buttonName)
    Rayfield:Notify({ Title = buttonName, Content = "Duping, please wait...", Duration = 3 })
    task.wait(3)
    local variations = {
        ["Glitter Bomb"]="Glitter Bomb", ["Ghostly Gloves"]="Ghostly Gloves",
        ["Scythe"]="Scythe", ["Spiked Knuckles"]="Spiked Knuckles",
        ["Ice Crown Queen"]="Ice Crown Queen", ["Trident"]="Trident",
        ["I
      
