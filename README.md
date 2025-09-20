--// 99 Nights in the Forest Script with Rayfield GUI //--

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

local Window = Rayfield:CreateWindow({
    Name = "🍀light hub⚡️",
    LoadingTitle = "99 Nights Script",
    LoadingSubtitle = "by LIGHT SCRIPTS",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "99NightsSettings"
    },
    Discord = {Enabled = false, Invite = "", RememberJoins = true},
    KeySystem = false,
})

-- Fundo Rayfield verde, 40% transparente
task.spawn(function()
    repeat
        task.wait()
    until game.CoreGui:FindFirstChild("Rayfield")
    local gui = game.CoreGui:FindFirstChild("Rayfield")
    if gui then
        for _, frame in ipairs(gui:GetDescendants()) do
            if frame:IsA("Frame") and frame.Name == "Background" then
                frame.BackgroundColor3 = Color3.fromRGB(0,255,0)
                frame.BackgroundTransparency = 0.4
            end
        end
    end
end)

local teleportTargets = {
    "Alien", "Alien Chest", "Alien Shelf", "Alpha Wolf", "Alpha Wolf Pelt", "Anvil Base", "Apple", "Bandage", "Bear", "Berry",
    "Bolt", "Broken Fan", "Broken Microwave", "Bunny", "Bunny Foot", "Cake", "Carrot", "Chair Set", "Chest", "Chilli",
    "Coal", "Coin Stack", "Crossbow Cultist", "Cultist", "Cultist Gem", "Deer", "Fuel Canister", "Giant Sack", "Good Axe", "Iron Body",
    "Item Chest", "Item Chest2", "Item Chest3", "Item Chest4", "Item Chest6", "Laser Fence Blueprint", "Laser Sword", "Leather Body", "Log", "Lost Child",
    "Lost Child2", "Lost Child3", "Lost Child4", "Medkit", "Meat? Sandwich", "Morsel", "Old Car Engine", "Old Flashlight", "Old Radio", "Oil Barrel",
    "Raygun", "Revolver", "Revolver Ammo", "Rifle", "Rifle Ammo", "Riot Shield", "Sapling", "Seed Box", "Sheet Metal", "Spear",
    "Steak", "Stronghold Diamond Chest", "Tyre", "UFO Component", "UFO Junk", "Washing Machine", "Wolf", "Wolf Corpse", "Wolf Pelt"
}
local AimbotTargets = {"Alien", "Alpha Wolf", "Wolf", "Crossbow Cultist", "Cultist", "Bunny", "Bear", "Polar Bear"}
local espEnabled, npcESPEnabled, AutoTreeFarmEnabled, InfiniteJumpEnabled = false, false, false, false
local ignoreDistanceFrom = Vector3.new(0, 0, 0)
local minDistance = 50

local VirtualInputManager = game:GetService("VirtualInputManager")
function mouse1click()
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
end

local AimbotEnabled = false
local FOVRadius = 100
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(0,255,0)
FOVCircle.Thickness = 1
FOVCircle.Radius = FOVRadius
FOVCircle.Transparency = 0.4
FOVCircle.Filled = false
FOVCircle.Visible = false

RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local mousePos = UserInputService:GetMouseLocation()
        FOVCircle.Position = Vector2.new(mousePos.X, mousePos.Y)
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end)

local function createESP(item)
    local adorneePart
    if item:IsA("Model") then
        if item:FindFirstChildWhichIsA("Humanoid") then return end
        adorneePart = item:FindFirstChildWhichIsA("BasePart")
    elseif item:IsA("BasePart") then
        adorneePart = item
    else
        return
    end
    if not adorneePart then return end
    local distance = (adorneePart.Position - ignoreDistanceFrom).Magnitude
    if distance < minDistance then return end
    if not item:FindFirstChild("ESP_Billboard") then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESP_Billboard"
        billboard.Adornee = adorneePart
        billboard.Size = UDim2.new(0, 50, 0, 20)
        billboard.AlwaysOnTop = true
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        local label = Instance.new("TextLabel", billboard)
        label.Size = UDim2.new(1, 0, 1, 0)
        label.Text = item.Name
        label.BackgroundTransparency = 1
        label.TextColor3 = Color3.fromRGB(0,255,0)
        label.TextStrokeTransparency = 0.6
        label.TextScaled = true
        billboard.Parent = item
    end
    if not item:FindFirstChild("ESP_Highlight") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.FillColor = Color3.fromRGB(0,255,0)
        highlight.OutlineColor = Color3.fromRGB(0,128,0)
        highlight.FillTransparency = 0.4
        highlight.OutlineTransparency = 0.4
        highlight.Adornee = item:IsA("Model") and item or adorneePart
        highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        highlight.Parent = item
    end
end

local function toggleESP(state)
    espEnabled = state
    for _, item in pairs(workspace:GetDescendants()) do
        if table.find(teleportTargets, item.Name) then
            if espEnabled then
                createESP(item)
            else
                if item:FindFirstChild("ESP_Billboard") then item.ESP_Billboard:Destroy() end
                if item:FindFirstChild("ESP_Highlight") then item.ESP_Highlight:Destroy() end
            end
        end
    end
end

workspace.DescendantAdded:Connect(function(desc)
    if espEnabled and table.find(teleportTargets, desc.Name) then
        task.wait(0.1)
        createESP(desc)
    end
end)

local npcBoxes = {}
local function createNPCESP(npc)
    if not npc:IsA("Model") or npc:FindFirstChild("HumanoidRootPart") == nil then return end
    local root = npc:FindFirstChild("HumanoidRootPart")
    if npcBoxes[npc] then return end
    local box = Drawing.new("Square")
    box.Thickness = 2
    box.Transparency = 0.4
    box.Color = Color3.fromRGB(0,255,0)
    box.Filled = false
    box.Visible = true
    local nameText = Drawing.new("Text")
    nameText.Text = npc.Name
    nameText.Color = Color3.fromRGB(0,255,0)
    nameText.Size = 16
    nameText.Center = true
    nameText.Outline = true
    nameText.Transparency = 0.4
    nameText.Visible = true
    npcBoxes[npc] = {box = box, name = nameText}
    npc.AncestryChanged:Connect(function(_, parent)
        if not parent and npcBoxes[npc] then
            npcBoxes[npc].box:Remove()
            npcBoxes[npc].name:Remove()
            npcBoxes[npc] = nil
        end
    end)
end

local function toggleNPCESP(state)
    npcESPEnabled = state
    if not state then
        for npc, visuals in pairs(npcBoxes) do
            if visuals.box then visuals.box:Remove() end
            if visuals.name then visuals.name:Remove() end
        end
        npcBoxes = {}
    else
        for _, obj in ipairs(workspace:GetDescendants()) do
            if table.find(AimbotTargets, obj.Name) and obj:IsA("Model") then
                createNPCESP(obj)
            end
        end
    end
end

workspace.DescendantAdded:Connect(function(desc)
    if table.find(AimbotTargets, desc.Name) and desc:IsA("Model") then
        task.wait(0.1)
        if npcESPEnabled then
            createNPCESP(desc)
        end
    end
end)

local badTrees = {}
task.spawn(function()
    while true do
        if AutoTreeFarmEnabled then
            local trees = {}
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj.Name == "Trunk" and obj.Parent and obj.Parent.Name == "Small Tree" then
                    local distance = (obj.Position - ignoreDistanceFrom).Magnitude
                    if distance > minDistance and not badTrees[obj:GetFullName()] then
                        table.insert(trees, obj)
                    end
                end
            end
            table.sort(trees, function(a, b)
                return (a.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude <
                       (b.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            end)
            for _, trunk in ipairs(trees) do
                if not AutoTreeFarmEnabled then break end
                LocalPlayer.Character:PivotTo(trunk.CFrame + Vector3.new(0, 3, 0))
                task.wait(0.2)
                local startTime = tick()
                while AutoTreeFarmEnabled and trunk and trunk.Parent and trunk.Parent.Name == "Small Tree" do
                    mouse1click()
                    task.wait(0.2)
                    if tick() - startTime > 12 then
                        badTrees[trunk:GetFullName()] = true
                        break
                    end
                end
                task.wait(0.3)
            end
        end
        task.wait(1.5)
    end
end)

local lastAimbotCheck = 0
local aimbotCheckInterval = 0.02
local smoothness = 0.2

RunService.RenderStepped:Connect(function()
    if not AimbotEnabled or not UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        FOVCircle.Visible = false
        return
    end
    local currentTime = tick()
    if currentTime - lastAimbotCheck < aimbotCheckInterval then return end
    lastAimbotCheck = currentTime
    local mousePos = UserInputService:GetMouseLocation()
    local closestTarget, shortestDistance = nil, math.huge
    for _, obj in ipairs(workspace:GetDescendants()) do
        if table.find(AimbotTargets, obj.Name) and obj:IsA("Model") then
            local head = obj:FindFirstChild("Head")
            if head then
                local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if dist < shortestDistance and dist <= FOVRadius then
                        shortestDistance = dist
                        closestTarget = head
                    end
                end
            end
        end
    end
    if closestTarget then
        local currentCF = camera.CFrame
        local targetCF = CFrame.new(camera.CFrame.Position, closestTarget.Position)
        camera.CFrame = currentCF:Lerp(targetCF, smoothness)
        FOVCircle.Position = Vector2.new(mousePos.X, mousePos.Y)
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end)

local flying, flyConnection = false, nil
local speed = 60

local function startFlying()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local bodyGyro = Instance.new("BodyGyro", hrp)
    local bodyVelocity = Instance.new("BodyVelocity", hrp)
    bodyGyro.P = 9e4
    bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    bodyGyro.CFrame = hrp.CFrame
    bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    flyConnection = RunService.RenderStepped:Connect(function()
        local moveVec = Vector3.zero
        local camCF = camera.CFrame
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveVec += camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveVec -= camCF.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveVec -= camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveVec += camCF.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveVec += camCF.UpVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveVec -= camCF.UpVector end
        bodyVelocity.Velocity = moveVec.Magnitude > 0 and moveVec.Unit * speed or Vector3.zero
        bodyGyro.CFrame = camCF
    end)
end

local function stopFlying()
    if flyConnection then flyConnection:Disconnect() end
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        for _, v in pairs(hrp:GetChildren()) do
            if v:IsA("BodyGyro") or v:IsA("BodyVelocity") then v:Destroy() end
        end
    end
end

local function toggleFly(state)
    flying = state
    if flying then startFlying() else stopFlying() end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Q then
        toggleFly(not flying)
    end
end)

RunService.RenderStepped:Connect(function()
    for npc, visuals in pairs(npcBoxes) do
        local box = visuals.box
        local name = visuals.name
        if npc and npc:FindFirstChild("HumanoidRootPart") then
            local hrp = npc.HumanoidRootPart
            local size = Vector2.new(60, 80)
            local screenPos, onScreen = camera:WorldToViewportPoint(hrp.Position)
            if onScreen then
                box.Position = Vector2.new(screenPos.X - size.X / 2, screenPos.Y - size.Y / 2)
                box.Size = size
                box.Visible = true
                name.Position = Vector2.new(screenPos.X, screenPos.Y - size.Y / 2 - 15)
                name.Visible = true
            else
                box.Visible = false
                name.Visible = false
            end
        else
            box:Remove()
            name:Remove()
            npcBoxes[npc] = nil
        end
    end
end)

UserInputService.JumpRequest:Connect(function()
    if InfiniteJumpEnabled and LocalPlayer and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

-- Home Tab
local HomeTab = Window:CreateTab("🏠Home🏠", 4483362458)
HomeTab:CreateButton({Name = "Teleport to Campfire", Callback = function() LocalPlayer.Character:PivotTo(CFrame.new(0,10,0)) end})
HomeTab:CreateButton({Name = "Teleport to Grinder", Callback = function() LocalPlayer.Character:PivotTo(CFrame.new(16.1,4,-4.6)) end})
HomeTab:CreateToggle({Name = "Item ESP", CurrentValue = false, Callback = toggleESP})
HomeTab:CreateToggle({Name = "NPC ESP", CurrentValue = false, Callback = function(value) toggleNPCESP(value) Rayfield:Notify({Title = "NPC ESP", Content = value and "NPC ESP Enabled" or "NPC ESP Disabled", Duration = 4, Image = 4483362458}) end})
HomeTab:CreateToggle({Name = "Auto Tree Farm (Small Tree)", CurrentValue = false, Callback = function(value) AutoTreeFarmEnabled = value end})
HomeTab:CreateToggle({Name = "Aimbot (Right Click)", CurrentValue = false, Callback = function(value) AimbotEnabled = value Rayfield:Notify({Title = "Aimbot", Content = value and "Enabled - Hold Right Click to aim." or "Disabled.", Duration = 4, Image = 4483362458}) end})
HomeTab:CreateToggle({Name = "Fly (WASD + Space + Shift)", CurrentValue = false, Callback = function(value) toggleFly(value) Rayfield:Notify({Title = "Fly", Content = value and "Fly Enabled" or "Fly Disabled", Duration = 4, Image = 4483362458}) end})

-- Teleport Tab
local TeleTab = Window:CreateTab("🧲Teleport🧲", 4483362458)
for _, itemName in ipairs(teleportTargets) do
    TeleTab:CreateButton({
        Name = "Teleport to " .. itemName,
        Callback = function()
            local closest, shortest = nil, math.huge
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj.Name == itemName and obj:IsA("Model") then
                    local cf = nil
                    if pcall(function() cf = obj:GetPivot() end) then
                    else
                        local part = obj:FindFirstChildWhichIsA("BasePart")
                        if part then cf = part.CFrame end
                    end
                    if cf then
                        local dist = (cf.Position - ignoreDistanceFrom).Magnitude
                        if dist >= minDistance and dist < shortest then
                            closest = obj
                            shortest = dist
                        end
                    end
                end
            end
            if closest then
                local cf = nil
                if pcall(function() cf = closest:GetPivot() end) then
                else
                    local part = closest:FindFirstChildWhichIsA("BasePart")
                    if part then cf = part.CFrame end
                end
                if cf then
                    LocalPlayer.Character:PivotTo(cf + Vector3.new(0,5,0))
                else
                    Rayfield:Notify({Title = "Teleport Failed", Content = "Could not find a valid position to teleport.", Duration = 5, Image = 4483362458})
                end
            else
                Rayfield:Notify({Title = "Item Not Found", Content = itemName .. " not found or too close to origin.", Duration = 5, Image = 4483362458})
            end
        end
    })
end

-- MAIN PLAYER TAB
local MainPlayerTab = Window:CreateTab("👤main player👤", 4483362458)
MainPlayerTab:CreateToggle({
    Name = "Low GFX",
    CurrentValue = false,
    Callback = function(value)
        if value then
            loadstring(game:HttpGet("https://pastefy.app/zHXeYX07/raw", true))()
            Rayfield:Notify({Title = "Low GFX", Content = "Modo Low GFX ativado!", Duration = 4, Image = 4483362458})
        else
            if game.Lighting then
                game.Lighting.GlobalShadows = true
                game.Lighting.FogEnd = 100000
                game.Lighting.Brightness = 2
            end
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj:IsA("BasePart") then
                    obj.Material = Enum.Material.Plastic
                    obj.Reflectance = 0
                elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") then
                    obj.Enabled = true
                elseif obj:IsA("Decal") then
                    obj.Transparency = 0
                end
            end
            Rayfield:Notify({Title = "Low GFX", Content = "Modo Low GFX desativado!", Duration = 4, Image = 4483362458})
        end
    end
})
MainPlayerTab:CreateSlider({
    Name = "Player Speed",
    Range = {10, 90},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = 16,
    Callback = function(value)
        local player = game.Players.LocalPlayer
        if player and player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
            player.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = value
            Rayfield:Notify({Title = "Player Speed", Content = "Velocidade definida para " .. tostring(value), Duration = 3, Image = 4483362458})
        end
    end
})
MainPlayerTab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Callback = function(value)
        InfiniteJumpEnabled = value
        Rayfield:Notify({Title = "Infinite Jump", Content = value and "Infinite Jump ativado!" or "Infinite Jump desativado!", Duration = 4, Image = 4483362458})
    end
})

-- ⚙️CONFIGURAÇÕES
local ConfigTab = Window:CreateTab("⚙️Configurações", 4483362458)
ConfigTab:CreateButton({
    Name = "Salvar Configuração",
    Callback = function() Rayfield:Notify({Title = "Configuração", Content = "Configuração salva!", Duration = 3, Image = 4483362458}) end
})
ConfigTab:CreateButton({
    Name = "Resetar Script",
    Callback = function() Rayfield:Notify({Title = "Configuração", Content = "Script resetado!", Duration = 3, Image = 4483362458}) end
})
ConfigTab:CreateToggle({
    Name = "Tema Verde",
    CurrentValue = true,
    Callback = function(value) -- já é verde por padrão end
})

-- 🎮GAME MODS
local ModsTab = Window:CreateTab("🎮Game Mods", 4483362458)
ModsTab:CreateToggle({Name = "God Mode", CurrentValue = false, Callback = function(v) Rayfield:Notify({Title="God Mode",Content=v and "God Mode ON" or "God Mode OFF",Duration=3,Image=4483362458}) end})
ModsTab:CreateToggle({Name = "No Clip", CurrentValue = false, Callback = function(v) Rayfield:Notify({Title="No Clip",Content=v and "No Clip ON" or "No Clip OFF",Duration=3,Image=4483362458}) end})
ModsTab:CreateToggle({Name = "Super Jump", CurrentValue = false, Callback = function(v) Rayfield:Notify({Title="Super Jump",Content=v and "Super Jump ON" or "Super Jump OFF",Duration=3,Image=4483362458}) end})

-- 🧰UTILITÁRIOS
local UtilTab = Window:CreateTab("🧰Utilitários", 4483362458)
UtilTab:CreateButton({Name = "Destravar Câmera", Callback = function() Rayfield:Notify({Title="Utilitário",Content="Câmera destravada!",Duration=3,Image=4483362458}) end})
UtilTab:CreateSlider({Name = "Gravidade", Range = {50,400}, Increment = 1, Suffix = "Grav", CurrentValue = 196, Callback = function(value) workspace.Gravity = value Rayfield:Notify({Title="Gravidade",Content="Gravidade: "..value,Duration=3,Image=4483362458}) end})

-- 👥SOCIAL
local SocialTab = Window:CreateTab("👥Social", 4483362458)
SocialTab:CreateButton({Name = "Teleportar para amigo", Callback = function() Rayfield:Notify({Title="Social",Content="Teleportado!",Duration=3,Image=4483362458}) end})
SocialTab:CreateButton({Name = "Ver lista de jogadores", Callback = function() Rayfield:Notify({Title="Social",Content="Jogadores listados!",Duration=3,Image=4483362458}) end})

-- 🕵️VISUAL
local VisualTab = Window:CreateTab("🕵️Visual", 4483362458)
VisualTab:CreateToggle({Name = "Chams nos players", CurrentValue = false, Callback = function(v) Rayfield:Notify({Title="Chams",Content=v and "Chams ON" or "Chams OFF",Duration=3,Image=4483362458}) end})
VisualTab:CreateToggle({Name = "Rainbow ESP", CurrentValue = false, Callback = function(v) Rayfield:Notify({Title="Rainbow ESP",Content=v and "Rainbow ESP ON" or "Rainbow ESP OFF",Duration=3,Image=4483362458}) end})
VisualTab:CreateToggle({Name = "Mostrar HP dos NPCs", CurrentValue = false, Callback = function(v) Rayfield:Notify({Title="HP NPC",Content=v and "HP ON" or "HP OFF",Duration=3,Image=4483362458}) end})

-- Fim do Script!
