-- ==========================================================
-- 🦆 鴨嘴獸腳本 - V3.4 終極全方位強化版 (Net Attack Edition)
-- ✅ 戰鬥：嵌入 Net 協議快速攻擊 (RegisterHit)
-- ✅ 功能：秒殺、ESP Pro、自動 V3/V4、全功能 UI
-- ✅ 規範：Y 鍵收起、右上灰叉刪除、滑鼠全屏拖拽
-- ==========================================================

local players = game:GetService("Players")
local rs = game:GetService("RunService")
local uis = game:GetService("UserInputService") 
local lp = players.LocalPlayer
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- [1. 初始化配置]
_G.DuckbillRunning = true 
_G.Config = {
    AutoHaki = false, AutoV3 = false, AutoV4 = false,
    FastAttack = false, OneKill = false, 
    Fly = false, FlySpeed = 2, 
    ESPEnabled = false, FPSBoost = false
}

local espStore = {}
local movingUp, movingDown = false, false
local flyBV, flyBG = nil, nil

-- [2. 核心功能邏輯]

-- A. 嵌入你要求的 Net 快速攻擊
task.spawn(function()
    local _ENV = (getgenv or getrenv or getfenv)()
    local Settings = {AutoClick = true, ClickDelay = 0}
    while _G.DuckbillRunning do
        task.wait(Settings.ClickDelay)
        if _G.Config.FastAttack then
            pcall(function()
                local Net = ReplicatedStorage:WaitForChild("Modules"):FindFirstChild("Net")
                local RegisterAttack = Net and Net:FindFirstChild("RE/RegisterAttack")
                local RegisterHit = Net and Net:FindFirstChild("RE/RegisterHit")
                local Enemies = workspace:FindFirstChild("Enemies")
                if RegisterAttack and RegisterHit then
                    local OthersEnemies = {}
                    for _, v in pairs(Enemies:GetChildren()) do
                        if v:FindFirstChildOfClass("Humanoid") and v:FindFirstChildOfClass("Humanoid").Health > 0 then
                            local parts = v:GetDescendants()
                            local validParts = {}
                            for _, part in ipairs(parts) do if part:IsA("BasePart") and part.CanCollide then table.insert(validParts, part) end end
                            local p = #validParts > 0 and validParts[math.random(1, #validParts)] or v:FindFirstChild("HumanoidRootPart")
                            if p and lp:DistanceFromCharacter(p.Position) < 25000 then table.insert(OthersEnemies, {v, p}) end
                        end
                    end
                    if #OthersEnemies > 0 then
                        RegisterAttack:FireServer(Settings.ClickDelay)
                        RegisterHit:FireServer(OthersEnemies[1][2], OthersEnemies)
                    end
                end
            end)
        end
    end
end)

-- B. 完美懸停飛行
task.spawn(function()
    while _G.DuckbillRunning do
        rs.Heartbeat:Wait()
        local char = lp.Character
        if _G.Config.Fly and char then
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hrp and hum then
                for _, v in pairs(char:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end
                hum.PlatformStand = true
                if not hrp:FindFirstChild("DuckFlyForce") then
                    flyBV = Instance.new("BodyVelocity", hrp); flyBV.Name = "DuckFlyForce"; flyBV.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                    flyBG = Instance.new("BodyGyro", hrp); flyBG.Name = "DuckFlyGyro"; flyBG.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
                end
                local velocity = hum.MoveDirection * (_G.Config.FlySpeed * 40)
                if movingUp then velocity = velocity + Vector3.new(0, 80, 0) elseif movingDown then velocity = velocity + Vector3.new(0, -80, 0) end
                flyBV.Velocity = velocity; flyBG.CFrame = workspace.CurrentCamera.CFrame
            end
        elseif char and char:FindFirstChild("HumanoidRootPart") then
            if char.HumanoidRootPart:FindFirstChild("DuckFlyForce") then char.HumanoidRootPart.DuckFlyForce:Destroy() end
            if char.HumanoidRootPart:FindFirstChild("DuckFlyGyro") then char.HumanoidRootPart.DuckFlyGyro:Destroy() end
            char:FindFirstChildOfClass("Humanoid").PlatformStand = false
        end
    end
end)

-- C. 自動化、秒殺與 ESP 資訊 (補回 OneKill)
task.spawn(function()
    while _G.DuckbillRunning do
        task.wait(0.3)
        local char = lp.Character
        if char then
            if _G.Config.AutoHaki and not char:FindFirstChild("HasBuso") then pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso") end) end
            if _G.Config.AutoV3 then pcall(function() ReplicatedStorage.Remotes.CommE:FireServer("ActivateAbility") end) end
            if _G.Config.AutoV4 then pcall(function() ReplicatedStorage.Remotes.CommF_:InvokeServer("Awaken") end) end
            if _G.Config.OneKill and char:FindFirstChild("HumanoidRootPart") then
                local hrp = char.HumanoidRootPart
                if hrp:FindFirstChild("SwordDanceAttack") or hrp:FindFirstChild("TridentDoughZ2") then
                    local old = hrp.CFrame; hrp.CFrame = CFrame.new(0, -100000, 0); task.wait(); hrp.CFrame = old
                end
            end
        end
        if _G.Config.ESPEnabled then
            for _, p in pairs(players:GetPlayers()) do
                if p ~= lp and p.Character and p.Character:FindFirstChild("Head") then
                    if not espStore[p] then
                        local b = Instance.new("BillboardGui", p.Character.Head); b.Size = UDim2.new(0, 100, 0, 70); b.AlwaysOnTop = true; b.StudsOffset = Vector3.new(0, 3, 0)
                        local t = Instance.new("TextLabel", b); t.Size = UDim2.new(1, 0, 1, 0); t.BackgroundTransparency = 1; t.TextColor3 = Color3.new(1, 0, 0); t.TextScaled = true; t.Font = "Code"
                        espStore[p] = t
                    end
                    local dist = math.floor((p.Character.PrimaryPart.Position - lp.Character.PrimaryPart.Position).Magnitude)
                    espStore[p].Text = p.DisplayName .. "\n[" .. dist .. "m]\nHP: " .. math.floor(p.Character.Humanoid.Health)
                end
            end
        else for p, v in pairs(espStore) do pcall(function() v.Parent:Destroy() end) end; espStore = {} end
    end
end)

-- [3. UI 介面注入] (補回消失的所有按鈕)
local target = (game:GetService("CoreGui"):FindFirstChild("RobloxGui") and game:GetService("CoreGui")) or lp.PlayerGui
if target:FindFirstChild("Duckbill_V34_Final") then target.Duckbill_V34_Final:Destroy() end
local sg = Instance.new("ScreenGui", target); sg.Name = "Duckbill_V34_Final"; sg.ResetOnSpawn = false

local Menu = Instance.new("Frame", sg); Menu.Size = UDim2.new(0, 320, 0, 520); Menu.Position = UDim2.fromScale(0.5, 0.5); Menu.AnchorPoint = Vector2.new(0.5, 0.5); Menu.BackgroundColor3 = Color3.new(1, 1, 1); Instance.new("UICorner", Menu)
local Title = Instance.new("TextLabel", Menu); Title.Size = UDim2.new(1, -40, 0, 45); Title.Text = " 🦆 鴨嘴獸腳本 V3.4 旗艦版"; Title.Font = "GothamBold"; Title.Position = UDim2.new(0, 10, 0, 0); Title.BackgroundTransparency = 1; Title.TextXAlignment = 0
local CloseBtn = Instance.new("TextButton", Menu); CloseBtn.Size = UDim2.new(0, 25, 0, 25); CloseBtn.Position = UDim2.new(1, -30, 0, 10); CloseBtn.Text = "×"; CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 200, 200); Instance.new("UICorner", CloseBtn)

-- 拖拽
local d=false;local s;local sp;Menu.InputBegan:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then d=true;s=i.Position;sp=Menu.Position end end)
uis.InputChanged:Connect(function(i) if d and i.UserInputType==Enum.UserInputType.MouseMovement then local delta=i.Position-s;Menu.Position=UDim2.new(sp.X.Scale,sp.X.Offset+delta.X,sp.Y.Scale,sp.Y.Offset+delta.Y) end end)
uis.InputEnded:Connect(function(i) if i.UserInputType==Enum.UserInputType.MouseButton1 then d=false end end)

local container = Instance.new("ScrollingFrame", Menu); container.Size = UDim2.new(1, -20, 1, -60); container.Position = UDim2.new(0, 10, 0, 50); container.BackgroundTransparency = 1; container.ScrollBarThickness = 0
Instance.new("UIListLayout", container).Padding = UDim.new(0, 7)

local function AddToggle(name, key, callback)
    local btn = Instance.new("TextButton", container); btn.Size = UDim2.new(0.95, 0, 0, 38); btn.Text = name; btn.BackgroundColor3 = Color3.new(1, 1, 1); Instance.new("UICorner", btn)
    btn.MouseButton1Click:Connect(function()
        _G.Config[key] = not _G.Config[key]
        if callback then callback(_G.Config[key]) end
        btn.BackgroundColor3 = _G.Config[key] and Color3.fromRGB(50, 180, 80) or Color3.new(1, 1, 1)
        btn.TextColor3 = _G.Config[key] and Color3.new(1, 1, 1) or Color3.new(0, 0, 0)
    end)
end

AddToggle("快速攻擊 (Net 強化)", "FastAttack")
AddToggle("秒殺攻擊 (特定技能)", "OneKill")
AddToggle("透視玩家 (ESP Pro)", "ESPEnabled")
AddToggle("穿牆飛行 (1上/2下)", "Fly")

local SpdBtn = Instance.new("TextButton", container); SpdBtn.Size = UDim2.new(0.95, 0, 0, 38); SpdBtn.Text = "飛行速度係數: " .. _G.Config.FlySpeed; SpdBtn.BackgroundColor3 = Color3.fromRGB(240, 240, 240); Instance.new("UICorner", SpdBtn)
SpdBtn.MouseButton1Click:Connect(function() _G.Config.FlySpeed = (_G.Config.FlySpeed >= 10) and 1 or _G.Config.FlySpeed + 1; SpdBtn.Text = "飛行速度係數: " .. _G.Config.FlySpeed end)

AddToggle("自動 V4 覺醒", "AutoV4")
AddToggle("自動 V3 技能", "AutoV3")
AddToggle("自動武裝色", "AutoHaki")
AddToggle("暴力 FPS 優化", "FPSBoost", function(s)
    if s then settings().Rendering.QualityLevel = 1; for _,v in pairs(game:GetDescendants()) do pcall(function() if v:IsA("Decal") or v:IsA("Texture") or v:IsA("ParticleEmitter") then v:Destroy() end end) end end
end)

uis.InputBegan:Connect(function(i, g)
    if not g then
        if i.KeyCode == Enum.KeyCode.Y then Menu.Visible = not Menu.Visible end
        if i.KeyCode == Enum.KeyCode.One then movingUp = true end
        if i.KeyCode == Enum.KeyCode.Two then movingDown = true end
    end
end)
uis.InputEnded:Connect(function(i)
    if i.KeyCode == Enum.KeyCode.One then movingUp = false end
    if i.KeyCode == Enum.KeyCode.Two then movingDown = false end
end)
CloseBtn.MouseButton1Click:Connect(function() sg:Destroy(); _G.DuckbillRunning = false end)
