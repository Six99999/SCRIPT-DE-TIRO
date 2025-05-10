--// Inicialização Rayfield
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

local Window = Rayfield:CreateWindow({
    Name = "SIX FPS",
    LoadingTitle = "Carregando SIX FPS...",
    LoadingSubtitle = "por @SIX",
    ConfigurationSaving = { Enabled = false },
    Discord = {
        Enabled = true,
        Invite = "fzkR5utV",
        RememberJoins = false
    },
    KeySystem = false,
})

--// Funções Gerais
local lp = game.Players.LocalPlayer
local cam = workspace.CurrentCamera

--// Criação das abas
local CombatTab = Window:CreateTab("Aimbot", 4483362458)
local ESPTab = Window:CreateTab("ESP", 4483362458)
local MiscTab = Window:CreateTab("Utilidades", 4483362458)
local InfoTab = Window:CreateTab("Discord", 4483362458)

--// ABA: AIMBOT
local aimbotEnabled = false
local fov = 50

CombatTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = false,
    Callback = function(Value)
        aimbotEnabled = Value
    end,
})

CombatTab:CreateSlider({
    Name = "FOV do Aimbot",
    Range = {10, 100},
    Increment = 1,
    CurrentValue = fov,
    Callback = function(Value)
        fov = Value
    end,
})

-- FOV Círculo
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 0, 0)
fovCircle.Thickness = 1
fovCircle.Radius = fov
fovCircle.Transparency = 1
fovCircle.NumSides = 100
fovCircle.Filled = false
fovCircle.Visible = false

game:GetService("RunService").RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)
    fovCircle.Radius = fov
    fovCircle.Visible = aimbotEnabled

    if not aimbotEnabled then return end

    local closest, shortest = nil, math.huge
    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= lp and plr.Team ~= lp.Team and plr.Character and plr.Character:FindFirstChild("Head") then
            local head = plr.Character.Head.Position
            local screenPos, onScreen = cam:WorldToViewportPoint(head)

            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(cam.ViewportSize.X/2, cam.ViewportSize.Y/2)).Magnitude
                if dist < fov then
                    local rayParams = RaycastParams.new()
                    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
                    rayParams.FilterDescendantsInstances = {lp.Character}

                    local rayResult = workspace:Raycast(cam.CFrame.Position, (head - cam.CFrame.Position).Unit * 500, rayParams)

                    if rayResult and rayResult.Instance and rayResult.Instance:IsDescendantOf(plr.Character) then
                        if dist < shortest then
                            shortest = dist
                            closest = plr
                        end
                    end
                end
            end
        end
    end

    if closest and closest.Character and closest.Character:FindFirstChild("Head") then
        cam.CFrame = CFrame.new(cam.CFrame.Position, closest.Character.Head.Position)
    end
end)

--// ABA: ESP
local espEnabled = false
local textSize = 15

local function getTeamColor(player)
    local teamName = player.Team and player.Team.Name or ""
    if teamName == "CV" then
        return Color3.fromRGB(255, 0, 0)
    elseif teamName == "BOPE" then
        return Color3.fromRGB(0, 100, 255)
    else
        return Color3.new(1, 1, 1)
    end
end

local function createESPForPlayer(player)
    if player == lp or player.Team == lp.Team then return end
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end

    local root = player.Character.HumanoidRootPart
    if root:FindFirstChild("ESP") then return end

    local espGui = Instance.new("BillboardGui", root)
    espGui.Name = "ESP"
    espGui.Size = UDim2.new(0, 200, 0, 40)
    espGui.AlwaysOnTop = true
    espGui.Adornee = root

    local nameLabel = Instance.new("TextLabel", espGui)
    nameLabel.Size = UDim2.new(1, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = player.Name .. " [".. math.floor((player.Character.HumanoidRootPart.Position - lp.Character.HumanoidRootPart.Position).Magnitude) .. "m]"
    nameLabel.TextColor3 = getTeamColor(player)
    nameLabel.TextSize = textSize
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextStrokeTransparency = 0.4

    local box = Instance.new("BoxHandleAdornment", root)
    box.Name = "BoxESP"
    box.Size = Vector3.new(4, 6, 2)
    box.Color3 = getTeamColor(player)
    box.AlwaysOnTop = true
    box.Adornee = root
    box.ZIndex = 10
    box.Transparency = 0
end

local function refreshESP()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local root = player.Character.HumanoidRootPart
            if root:FindFirstChild("ESP") then root.ESP:Destroy() end
            if root:FindFirstChild("BoxESP") then root.BoxESP:Destroy() end
        end
    end
    for _, player in pairs(game.Players:GetPlayers()) do
        createESPForPlayer(player)
    end
end

local function enableESP()
    espEnabled = true
    refreshESP()

    game.Players.PlayerAdded:Connect(function(plr)
        plr.CharacterAdded:Connect(function()
            wait(1)
            if espEnabled then refreshESP() end
        end)
    end)

    lp.CharacterAdded:Connect(function()
        wait(1)
        if espEnabled then refreshESP() end
    end)

    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= lp then
            plr.CharacterAdded:Connect(function()
                wait(1)
                if espEnabled then refreshESP() end
            end)
        end
    end
end

local function disableESP()
    espEnabled = false
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local root = player.Character.HumanoidRootPart
            if root:FindFirstChild("ESP") then root.ESP:Destroy() end
            if root:FindFirstChild("BoxESP") then root.BoxESP:Destroy() end
        end
    end
end

ESPTab:CreateToggle({
    Name = "Ativar ESP por Time",
    CurrentValue = false,
    Callback = function(Value)
        if Value then enableESP() else disableESP() end
    end,
})

ESPTab:CreateSlider({
    Name = "Tamanho do Nome ESP",
    Range = {10, 30},
    Increment = 1,
    CurrentValue = 15,
    Callback = function(Value)
        textSize = Value
        if espEnabled then refreshESP() end
    end,
})

--// ABA: Fly
MiscTab:CreateButton({
    Name = "Ativar Fly",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"))()
    end,
})

--// ABA: Velocidade
local speed = 16
local humanoid = lp.Character and lp.Character:FindFirstChildOfClass("Humanoid")

lp.CharacterAdded:Connect(function(char)
    wait(1)
    humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = speed
    end
end)

MiscTab:CreateSlider({
    Name = "Velocidade",
    Range = {16, 100},
    Increment = 1,
    CurrentValue = 16,
    Callback = function(Value)
        speed = Value
        if humanoid then
            humanoid.WalkSpeed = speed
        end
    end,
})

--// Discord
InfoTab:CreateButton({
    Name = "Entrar no Discord",
    Callback = function()
        setclipboard("https://discord.gg/fzkR5utV")
    end,
})
