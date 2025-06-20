-- 👑 KING HUB – Blox Fruits | SCRIPT FINAL COMPLETO
getgenv().KING_HUB = {
    autoFarmLevel = false,
    autoHaki = true,
    autoChest = false,
    fruitDetector = false,
    safeMode = true,
    antiKick = true
}

local plr = game.Players.LocalPlayer
local ws = game:GetService("Workspace")
local http = game:GetService("HttpService")

-- NPCs por Level (exemplos, adicione mais se quiser)
local npcData = {
    {name = "Bandit Quest Giver", min = 1, max = 14, pos = CFrame.new(1060, 16, 1547)},
    {name = "Monkey Quest Giver", min = 15, max = 29, pos = CFrame.new(-1602, 8, 152)},
    {name = "Pirate Quest Giver", min = 30, max = 59, pos = CFrame.new(-4860, 20, 4308)},
    -- Adicione mais NPCs aqui se quiser
}

-- Anti Kick
if KING_HUB.antiKick then
    plr.Idled:Connect(function()
        local vu = game:GetService("VirtualUser")
        vu:Button2Down(Vector2.new(0,0), ws.CurrentCamera.CFrame)
        wait(1)
        vu:Button2Up(Vector2.new(0,0), ws.CurrentCamera.CFrame)
    end)
end

-- Funções básicas
local function safeTp(cf)
    if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
        plr.Character.HumanoidRootPart.CFrame = cf + Vector3.new(0,3,0)
        wait(0.5)
    end
end

local function enableHaki()
    if KING_HUB.autoHaki then
        pcall(function()
            if not plr.Character:FindFirstChild("HasBuso") then
                local buso = plr.Backpack:FindFirstChild("Buso Haki") or plr.Character:FindFirstChild("Buso Haki")
                if buso then buso:Activate() end
            end
        end)
    end
end

local function getQuestNPC()
    local lvl = plr.Data.Level.Value
    for _, npc in pairs(npcData) do
        if lvl >= npc.min and lvl <= npc.max then
            return npc
        end
    end
    return nil
end

local function acceptQuestAuto(npcInfo)
    safeTp(npcInfo.pos)
    for _, v in pairs(ws:GetDescendants()) do
        if v.Name == npcInfo.name and v:FindFirstChild("Head") then
            fireclickdetector(v.Head:FindFirstChildOfClass("ClickDetector"))
            wait(1)
            local gui = plr.PlayerGui:FindFirstChild("QuestChoose")
            if gui then
                for _, btn in pairs(gui:GetDescendants()) do
                    if btn:IsA("TextButton") then
                        btn:Activate()
                        break
                    end
                end
            end
            break
        end
    end
end

local function huntMobs()
    local questGui = plr.PlayerGui:FindFirstChild("Quest")
    if questGui then
        for _, mob in pairs(ws.Enemies:GetChildren()) do
            if mob:FindFirstChild("HumanoidRootPart") and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                repeat
                    enableHaki()
                    plr.Character.HumanoidRootPart.CFrame = mob.HumanoidRootPart.CFrame + Vector3.new(0,5,0)
                    local tool = plr.Character:FindFirstChildOfClass("Tool")
                    if tool then tool:Activate() end
                    wait(0.2)
                until mob.Humanoid.Health <= 0 or not KING_HUB.autoFarmLevel
            end
        end
    end
end

local function collectChests()
    for _, chest in pairs(ws:GetChildren()) do
        if chest:IsA("Model") and chest:FindFirstChild("HumanoidRootPart") and chest.Name:lower():find("chest") then
            safeTp(chest.HumanoidRootPart.CFrame)
            wait(0.5)
        end
    end
end

local function fruitFinder()
    for _, v in pairs(ws:GetDescendants()) do
        if v:IsA("Tool") and v.Name:lower():find("fruit") then
            safeTp(v.Handle.CFrame)
            break
        end
    end
end

local function safeModeCheck()
    if plr.Character and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health <= 0 then
        local placeId = game.PlaceId
        local servers = {}
        local req = syn and syn.request or http_request
        if req then
            local body = req({
                Url = string.format("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100", placeId),
                Method = "GET"
            })
            local data = http:JSONDecode(body.Body)
            for i,v in next, data.data do
                if v.playing < v.maxPlayers then
                    table.insert(servers,v.id)
                end
            end
            if #servers > 0 then
                game:GetService("TeleportService"):TeleportToPlaceInstance(placeId, servers[math.random(1,#servers)])
            end
        end
    end
end

-- GUI
local function createGui()
    local gui = Instance.new("ScreenGui", game.CoreGui)
    gui.Name = "KING_HUB_GUI"

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 300, 0, 200)
    frame.Position = UDim2.new(0, 10, 0, 100)
    frame.BackgroundColor3 = Color3.fromRGB(35,35,35)
    frame.Active = true
    frame.Draggable = true

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1, 0, 0, 30)
    title.Text = "👑 KING HUB – Blox Fruits"
    title.TextColor3 = Color3.new(1,1,1)
    title.BackgroundColor3 = Color3.fromRGB(50,50,50)
    title.TextScaled = true

    local farmBtn = Instance.new("TextButton", frame)
    farmBtn.Size = UDim2.new(1, -10, 0, 30)
    farmBtn.Position = UDim2.new(0, 5, 0, 40)
    farmBtn.Text = "Auto Farm Level: OFF"
    farmBtn.TextColor3 = Color3.new(1,1,1)
    farmBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    farmBtn.MouseButton1Click:Connect(function()
        KING_HUB.autoFarmLevel = not KING_HUB.autoFarmLevel
        farmBtn.Text = "Auto Farm Level: " .. (KING_HUB.autoFarmLevel and "ON" or "OFF")
    end)

    local chestBtn = Instance.new("TextButton", frame)
    chestBtn.Size = UDim2.new(1, -10, 0, 30)
    chestBtn.Position = UDim2.new(0, 5, 0, 80)
    chestBtn.Text = "Auto Chest: OFF"
    chestBtn.TextColor3 = Color3.new(1,1,1)
    chestBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    chestBtn.MouseButton1Click:Connect(function()
        KING_HUB.autoChest = not KING_HUB.autoChest
        chestBtn.Text = "Auto Chest: " .. (KING_HUB.autoChest and "ON" or "OFF")
    end)

    local fruitBtn = Instance.new("TextButton", frame)
    fruitBtn.Size = UDim2.new(1, -10, 0, 30)
    fruitBtn.Position = UDim2.new(0, 5, 0, 120)
    fruitBtn.Text = "Fruit Detector: OFF"
    fruitBtn.TextColor3 = Color3.new(1,1,1)
    fruitBtn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    fruitBtn.MouseButton1Click:Connect(function()
        KING_HUB.fruitDetector = not KING_HUB.fruitDetector
        fruitBtn.Text = "Fruit Detector: " .. (KING_HUB.fruitDetector and "ON" or "OFF")
    end)
end

-- LOOP
spawn(function()
    while wait(0.5) do
        if KING_HUB.autoFarmLevel then
            local npc = getQuestNPC()
            if npc then
                if not plr.PlayerGui:FindFirstChild("Quest") then
                    acceptQuestAuto(npc)
                else
                    huntMobs()
                end
            end
        end
        if KING_HUB.autoChest then collectChests() end
        if KING_HUB.fruitDetector then fruitFinder() end
        if KING_HUB.safeMode then safeModeCheck() end
    end
end)

createGui()
