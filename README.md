--// 99 Nights in the Forest Script with Orion UI //--

-- Load Orion UI Library
local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source"))()

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configurar tema verde
local greenTheme = {
    Main = Color3.fromRGB(0, 150, 0),
    Background = Color3.fromRGB(20, 40, 20),
    Accent = Color3.fromRGB(0, 100, 0),
    LightContrast = Color3.fromRGB(40, 80, 40),
    DarkContrast = Color3.fromRGB(10, 30, 10),
    TextColor = Color3.fromRGB(200, 255, 200)
}

-- Window Setup
local Window = OrionLib:MakeWindow({
    Name = "🍀light hub⚡️",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "99NightsSettings",
    IntroEnabled = false,
    Theme = greenTheme
})

-- Variables
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
local espEnabled = false
local npcESPEnabled = false
local ignoreDistanceFrom = Vector3.new(0, 0, 0)
local minDistance = 50
local AutoTreeFarmEnabled = false

-- Click simulation
local VirtualInputManager = game:GetService("VirtualInputManager")
function mouse1click()
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
end

-- Aimbot FOV Circle
local AimbotEnabled = false
local FOVRadius = 100
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(0, 255, 0)  -- Verde
FOVCircle.Thickness = 1
FOVCircle.Radius = FOVRadius
FOVCircle.Transparency = 0.5
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

-- ESP Function
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
        label.BackgroundTransparency = 0.9
        label.TextColor3 = Color3.fromRGB(0, 255, 0)  -- Verde
        label.TextStrokeTransparency = 0
        label.TextScaled = true
        billboard.Parent = item
    end

    if not item:FindFirstChild("ESP_Highlight") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.FillColor = Color3.fromRGB(0, 200, 0)  -- Verde
        highlight.OutlineColor = Color3.fromRGB(0, 100, 0)  -- Verde escuro
        highlight.FillTransparency = 0.25
        highlight.OutlineTransparency = 0
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

-- ESP for NPCs
local npcBoxes = {}

local function createNPCESP(npc)
    if not npc:IsA("Model") or npc:FindFirstChild("HumanoidRootPart") == nil then return end

    local root = npc:FindFirstChild("HumanoidRootPart")
    if npcBoxes[npc] then return end

    local box = Drawing.new("Square")
    box.Thickness = 2
    box.Transparency = 1
    box.Color = Color3.fromRGB(0, 255, 0)  -- Verde
    box.Filled = false
    box.Visible = true

    local nameText = Drawing.new("Text")
    nameText.Text = npc.Name
    nameText.Color = Color3.fromRGB(0, 255, 0)  -- Verde
    nameText.Size = 16
    nameText.Center = true
    nameText.Outline = true
    nameText.Visible = true

    npcBoxes[npc] = {box = box, name = nameText}

    -- Cleanup on remove
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
        -- Show NPC ESP for already existing NPCs
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

-- Auto Tree Farm Logic with timeout
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

-- Optimized Aimbot Logic
local lastAimbotCheck = 0
local aimbotCheckInterval = 0.02 -- Faster reaction time
local smoothness = 0.2 -- Smooth camera interpolation

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
        camera.CFrame = currentCF:Lerp(targetCF, smoothness) -- Smoothly rotate camera
        FOVCircle.Position = Vector2.new(mousePos.X, mousePos.Y)
        FOVCircle.Visible = true
    else
        FOVCircle.Visible = false
    end
end)

-- Fly Logic
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

-- GUI Tabs
local HomeTab = Window:MakeTab({
    Name = "🏠Home🏠",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

HomeTab:AddButton({
    Name = "Teleport to Campfire",
    Callback = function()
        LocalPlayer.Character:PivotTo(CFrame.new(0, 10, 0))
        OrionLib:MakeNotification({
            Name = "Teleport",
            Content = "Teleported to Campfire!",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

HomeTab:AddButton({
    Name = "Teleport to Grinder",
    Callback = function()
        LocalPlayer.Character:PivotTo(CFrame.new(16.1,4,-4.6))
        OrionLib:MakeNotification({
            Name = "Teleport",
            Content = "Teleported to Grinder!",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

HomeTab:AddToggle({
    Name = "Item ESP",
    Default = false,
    Callback = function(Value)
        toggleESP(Value)
        OrionLib:MakeNotification({
            Name = "Item ESP",
            Content = Value and "Item ESP Enabled" or "Item ESP Disabled",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

HomeTab:AddToggle({
    Name = "NPC ESP",
    Default = false,
    Callback = function(Value)
        toggleNPCESP(Value)
        OrionLib:MakeNotification({
            Name = "NPC ESP",
            Content = Value and "NPC ESP Enabled" or "NPC ESP Disabled",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

HomeTab:AddToggle({
    Name = "Auto Tree Farm (Small Tree)",
    Default = false,
    Callback = function(Value)
        AutoTreeFarmEnabled = Value
        OrionLib:MakeNotification({
            Name = "Auto Tree Farm",
            Content = Value and "Auto Tree Farm Enabled" or "Auto Tree Farm Disabled",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

HomeTab:AddToggle({
    Name = "Aimbot (Right Click)",
    Default = false,
    Callback = function(Value)
        AimbotEnabled = Value
        OrionLib:MakeNotification({
            Name = "Aimbot",
            Content = Value and "Enabled - Hold Right Click to aim." or "Disabled.",
            Image = "rbxassetid://4483362458",
            Time = 4
        })
    end
})

HomeTab:AddToggle({
    Name = "Fly (WASD + Space + Shift)",
    Default = false,
    Callback = function(Value)
        toggleFly(Value)
        OrionLib:MakeNotification({
            Name = "Fly",
            Content = Value and "Fly Enabled" or "Fly Disabled",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

-- Teleport Tab
local TeleTab = Window:MakeTab({
    Name = "🧲Teleport🧲",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

for _, itemName in ipairs(teleportTargets) do
    TeleTab:AddButton({
        Name = "Teleport to " .. itemName,
        Callback = function()
            local closest, shortest = nil, math.huge
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj.Name == itemName and obj:IsA("Model") then
                    local cf = nil
                    if pcall(function() cf = obj:GetPivot() end) then
                        -- success
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
                    -- success
                else
                    local part = closest:FindFirstChildWhichIsA("BasePart")
                    if part then cf = part.CFrame end
                end
                if cf then
                    LocalPlayer.Character:PivotTo(cf + Vector3.new(0, 5, 0))
                    OrionLib:MakeNotification({
                        Name = "Teleport",
                        Content = "Teleported to " .. itemName,
                        Image = "rbxassetid://4483362458",
                        Time = 3
                    })
                else
                    OrionLib:MakeNotification({
                        Name = "Teleport Failed",
                        Content = "Could not find a valid position to teleport.",
                        Image = "rbxassetid://4483362458",
                        Time = 5
                    })
                end
            else
                OrionLib:MakeNotification({
                    Name = "Item Not Found",
                    Content = itemName .. " not found or too close to origin.",
                    Image = "rbxassetid://4483362458",
                    Time = 5
                })
            end
        end
    })
end

local MainPlayerTab = Window:MakeTab({
    Name = "👤Main Player👤",
    Icon = "rbxassetid://4483362458",
    PremiumOnly = false
})

MainPlayerTab:AddToggle({
    Name = "Low GFX",
    Default = false,
    Callback = function(Value)
        if Value then
            loadstring(game:HttpGet("https://pastefy.app/zHXeYX07/raw", true))()
            OrionLib:MakeNotification({
                Name = "Low GFX",
                Content = "Modo Low GFX ativado!",
                Image = "rbxassetid://4483362458",
                Time = 4
            })
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

            OrionLib:MakeNotification({
                Name = "Low GFX",
                Content = "Modo Low GFX desativado!",
                Image = "rbxassetid://4483362458",
                Time = 4
            })
        end
    end
})

MainPlayerTab:AddSlider({
    Name = "Player Speed",
    Min = 10,
    Max = 90,
    Default = 16,
    Color = Color3.fromRGB(0, 150, 0),  -- Verde
    Increment = 1,
    ValueName = "Speed",
    Callback = function(Value)
        local player = game.Players.LocalPlayer
        if player and player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
            player.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = Value
            OrionLib:MakeNotification({
                Name = "Player Speed",
                Content = "Velocidade definida para " .. tostring(Value),
                Image = "rbxassetid://4483362458",
                Time = 3
            })
        end
    end
})

-- Inicializar Orion
OrionLib:Init()