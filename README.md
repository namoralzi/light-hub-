--// 99 Nights in the Forest Script with Orion GUI //--

loadstring(game:HttpGet("https://raw.githubusercontent.com/jensonhirst/Orion/main/source"))()

local OrionLib = loadstring(game:HttpGet("https://raw.githubusercontent.com/jensonhirst/Orion/main/source"))()

-- Escolha a cor do tema! Exemplos: "Default", "Dark", "Light", "Red", "Green", "Blue", "Yellow"
local themeColor = "Green" -- você pode trocar para outra cor!

local Window = OrionLib:MakeWindow({
    Name = "🍀light hub⚡️",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "99NightsOrion",
    IntroEnabled = true,
    IntroText = "by LIGHT SCRIPTS",
    Theme = themeColor
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local camera = workspace.CurrentCamera

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
local InfiniteJumpEnabled = false
local AimbotEnabled = false
local flying = false

local VirtualInputManager = game:GetService("VirtualInputManager")
function mouse1click()
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
    VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
end

-- ESP Functions (Billboard/Highlight)
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

-- NPC ESP (Billboard only, Drawing not suported in Orion)
local function toggleNPCESP(state)
    npcESPEnabled = state
    for _, item in pairs(workspace:GetDescendants()) do
        if table.find(AimbotTargets, item.Name) and item:IsA("Model") and item:FindFirstChild("HumanoidRootPart") then
            if state then
                if not item:FindFirstChild("NPCESP_Billboard") then
                    local billboard = Instance.new("BillboardGui", item.HumanoidRootPart)
                    billboard.Name = "NPCESP_Billboard"
                    billboard.Adornee = item.HumanoidRootPart
                    billboard.Size = UDim2.new(0, 100, 0, 20)
                    billboard.AlwaysOnTop = true
                    billboard.StudsOffset = Vector3.new(0, 2, 0)
                    local label = Instance.new("TextLabel", billboard)
                    label.Size = UDim2.new(1, 0, 1, 0)
                    label.Text = item.Name
                    label.BackgroundTransparency = 1
                    label.TextColor3 = Color3.fromRGB(0,255,0)
                    label.TextScaled = true
                    label.TextStrokeTransparency = 0.6
                end
            else
                if item.HumanoidRootPart:FindFirstChild("NPCESP_Billboard") then
                    item.HumanoidRootPart.NPCESP_Billboard:Destroy()
                end
            end
        end
    end
end

workspace.DescendantAdded:Connect(function(desc)
    if npcESPEnabled and table.find(AimbotTargets, desc.Name) and desc:IsA("Model") and desc:FindFirstChild("HumanoidRootPart") then
        task.wait(0.1)
        if not desc:FindFirstChild("NPCESP_Billboard") then
            local billboard = Instance.new("BillboardGui", desc.HumanoidRootPart)
            billboard.Name = "NPCESP_Billboard"
            billboard.Adornee = desc.HumanoidRootPart
            billboard.Size = UDim2.new(0, 100, 0, 20)
            billboard.AlwaysOnTop = true
            billboard.StudsOffset = Vector3.new(0, 2, 0)
            local label = Instance.new("TextLabel", billboard)
            label.Size = UDim2.new(1, 0, 1, 0)
            label.Text = desc.Name
            label.BackgroundTransparency = 1
            label.TextColor3 = Color3.fromRGB(0,255,0)
            label.TextScaled = true
            label.TextStrokeTransparency = 0.6
        end
    end
end)

-- Auto Tree Farm
spawn(function()
    local badTrees = {}
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
                wait(0.2)
                local startTime = tick()
                while AutoTreeFarmEnabled and trunk and trunk.Parent and trunk.Parent.Name == "Small Tree" do
                    mouse1click()
                    wait(0.2)
                    if tick() - startTime > 12 then
                        badTrees[trunk:GetFullName()] = true
                        break
                    end
                end
                wait(0.3)
            end
        end
        wait(1.5)
    end
end)

-- Fly
local flyConnection = nil
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

-- Infinite Jump
UserInputService.JumpRequest:Connect(function()
    if InfiniteJumpEnabled and LocalPlayer and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
        LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

-- Orion Tabs
local HomeTab = Window:MakeTab({Name = "🏠Home🏠", Icon = "rbxassetid://4483362458"})
HomeTab:AddButton({
    Name = "Teleport to Campfire",
    Callback = function()
        LocalPlayer.Character:PivotTo(CFrame.new(0, 10, 0))
    end
})
HomeTab:AddButton({
    Name = "Teleport to Grinder",
    Callback = function()
        LocalPlayer.Character:PivotTo(CFrame.new(16.1,4,-4.6))
    end
})
HomeTab:AddToggle({
    Name = "Item ESP",
    Default = false,
    Callback = toggleESP
})
HomeTab:AddToggle({
    Name = "NPC ESP",
    Default = false,
    Callback = function(value)
        toggleNPCESP(value)
        OrionLib:MakeNotification({
            Name = "NPC ESP",
            Content = value and "NPC ESP Enabled" or "NPC ESP Disabled",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})
HomeTab:AddToggle({
    Name = "Auto Tree Farm (Small Tree)",
    Default = false,
    Callback = function(value)
        AutoTreeFarmEnabled = value
    end
})
HomeTab:AddToggle({
    Name = "Fly (WASD + Space + Shift)",
    Default = false,
    Callback = function(value)
        toggleFly(value)
        OrionLib:MakeNotification({
            Name = "Fly",
            Content = value and "Fly Enabled" or "Fly Disabled",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

-- Teleport Tab
local TeleTab = Window:MakeTab({Name = "🧲Teleport🧲", Icon = "rbxassetid://4483362458"})
for _, itemName in ipairs(teleportTargets) do
    TeleTab:AddButton({
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
                    OrionLib:MakeNotification({
                        Name = "Teleport Failed",
                        Content = "Could not find a valid position to teleport.",
                        Image = "rbxassetid://4483362458",
                        Time = 4
                    })
                end
            else
                OrionLib:MakeNotification({
                    Name = "Item Not Found",
                    Content = itemName .. " not found or too close to origin.",
                    Image = "rbxassetid://4483362458",
                    Time = 4
                })
            end
        end
    })
end

-- MAIN PLAYER TAB
local MainPlayerTab = Window:MakeTab({Name = "👤main player👤", Icon = "rbxassetid://4483362458"})
MainPlayerTab:AddToggle({
    Name = "Low GFX",
    Default = false,
    Callback = function(value)
        if value then
            loadstring(game:HttpGet("https://pastefy.app/zHXeYX07/raw", true))()
            OrionLib:MakeNotification({
                Name = "Low GFX",
                Content = "Modo Low GFX ativado!",
                Image = "rbxassetid://4483362458",
                Time = 3
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
                Time = 3
            })
        end
    end
})
MainPlayerTab:AddSlider({
    Name = "Player Speed",
    Min = 10,
    Max = 90,
    Default = 16,
    Increment = 1,
    ValueName = "Speed",
    Callback = function(value)
        if LocalPlayer and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").WalkSpeed = value
            OrionLib:MakeNotification({
                Name = "Player Speed",
                Content = "Velocidade definida para " .. tostring(value),
                Image = "rbxassetid://4483362458",
                Time = 3
            })
        end
    end
})
MainPlayerTab:AddToggle({
    Name = "Infinite Jump",
    Default = false,
    Callback = function(value)
        InfiniteJumpEnabled = value
        OrionLib:MakeNotification({
            Name = "Infinite Jump",
            Content = value and "Infinite Jump ativado!" or "Infinite Jump desativado!",
            Image = "rbxassetid://4483362458",
            Time = 3
        })
    end
})

OrionLib:Init()
