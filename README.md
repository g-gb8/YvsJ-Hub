local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "🥞 Garfield HUB | YvsJ & BEAR",
   Icon = 4483362458,
   LoadingTitle = "Carregando...",
   LoadingSubtitle = "Comer lasanha & Fugir do Bear 🐻",
   ConfigurationSaving = { Enabled = false },
   Discord = { Enabled = false }
})

-- === VARIÁVEIS ===
local Player = game.Players.LocalPlayer
local Lighting = game:GetService("Lighting")
local UIS = game:GetService("UserInputService")
local RS = game:GetService("RunService")
local Players = game:GetService("Players")
local VU = game:GetService("VirtualUser")

local FullBrightEnabled = false
local GhostModeEnabled = false
local ESPEnabled = false
local ChamsEnabled = false
local InfiniteJumpEnabled = false
local AntiAFKEnabled = false
local SavedPosition = nil
local espBoxes = {}
local espHighlights = {}
local noclipConnection = nil
local defaultLighting = {}

-- Salva defaults do Lighting
for _, effect in pairs(Lighting:GetChildren()) do
    if effect:IsA("PostEffect") then
        defaultLighting[effect.Name] = {Enabled = effect.Enabled}
    end
end
defaultLighting.Brightness = Lighting.Brightness
defaultLighting.ClockTime = Lighting.ClockTime
defaultLighting.GlobalShadows = Lighting.GlobalShadows
defaultLighting.FogEnd = Lighting.FogEnd
defaultLighting.Ambient = Lighting.Ambient

-- === FUNÇÕES ESP ===
local function ClearESP()
    for _, box in pairs(espBoxes) do if box then box:Destroy() end end
    for _, hl in pairs(espHighlights) do if hl then hl:Destroy() end end
    espBoxes = {}
    espHighlights = {}
end

local function createESP(character, color)
    local root = character:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local box = Instance.new("BoxHandleAdornment")
    box.Name = "ESPBox"
    box.Adornee = root
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(4, 6, 2)
    box.Color3 = color
    box.Transparency = 0.6
    box.Parent = root
    espBoxes[character] = box

    local hl = Instance.new("Highlight")
    hl.Adornee = character
    hl.FillColor = color
    hl.OutlineColor = color
    hl.FillTransparency = 0.35
    hl.OutlineTransparency = 0.1
    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Parent = character
    espHighlights[character] = hl
end

local function updateESP()
    if not (ESPEnabled or ChamsEnabled) then return end
    ClearESP()

    for _, v in pairs(Players:GetPlayers()) do
        if v \~= Player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local color = v.Team == Player.Team and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
            createESP(v.Character, color)
        end
    end
end

-- === ABA PRINCIPAL ===
local TabMain = Window:CreateTab("Principal", 4483362458)

TabMain:CreateSection("Visão 👁️")

TabMain:CreateToggle({
    Name = "ESP (Caixas 3D)",
    CurrentValue = false,
    Callback = function(Value)
        ESPEnabled = Value
        if not Value then ClearESP() else task.spawn(updateESP) end
    end,
})

TabMain:CreateToggle({
    Name = "Chams (Highlight Corpo)",
    CurrentValue = false,
    Callback = function(Value)
        ChamsEnabled = Value
        if not Value then ClearESP() else task.spawn(updateESP) end
    end,
})

TabMain:CreateToggle({
    Name = "Ghost Mode (Noclip)",
    CurrentValue = false,
    Callback = function(Value)
        GhostModeEnabled = Value
        if Value then
            noclipConnection = RS.Stepped:Connect(function()
                if Player.Character then
                    for _, part in pairs(Player.Character:GetDescendants()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                end
            end)
        else
            if noclipConnection then noclipConnection:Disconnect() noclipConnection = nil end
        end
    end,
})

TabMain:CreateSection("Visual")

TabMain:CreateToggle({
    Name = "Full Bright + No Fog",
    CurrentValue = false,
    Callback = function(Value)
        FullBrightEnabled = Value
        if Value then
            for _, v in pairs(Lighting:GetChildren()) do
                if v:IsA("PostEffect") then v.Enabled = false end
            end
            Lighting.Brightness = 5
            Lighting.ClockTime = 14
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 999999
            Lighting.Ambient = Color3.fromRGB(200, 200, 200)
        else
            Lighting.Brightness = defaultLighting.Brightness or 1
            Lighting.ClockTime = defaultLighting.ClockTime or 14
            Lighting.GlobalShadows = defaultLighting.GlobalShadows or true
            Lighting.FogEnd = defaultLighting.FogEnd or 100000
            Lighting.Ambient = defaultLighting.Ambient or Color3.new(0,0,0)
            for name, data in pairs(defaultLighting) do
                local effect = Lighting:FindFirstChild(name)
                if effect and data.Enabled \~= nil then effect.Enabled = data.Enabled end
            end
        end
    end,
})

TabMain:CreateSection("Movimento")

local speedValue = 16
TabMain:CreateSlider({
    Name = "WalkSpeed",
    Range = {16, 120},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = 16,
    Callback = function(Value)
        speedValue = Value
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = Value
        end
    end,
})

TabMain:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Callback = function(Value)
        InfiniteJumpEnabled = Value
    end,
})

UIS.JumpRequest:Connect(function()
    if InfiniteJumpEnabled and Player.Character then
        Player.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

TabMain:CreateSection("Teleporte")

TabMain:CreateButton({
   Name = "Salvar Posição Atual",
   Callback = function()
       if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
           SavedPosition = Player.Character.HumanoidRootPart.CFrame
           Rayfield:Notify({Title = "Sucesso", Content = "Posição salva!", Duration = 3})
       end
   end,
})

TabMain:CreateButton({
   Name = "Ir para Posição Salva",
   Callback = function()
       if SavedPosition and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
           Player.Character.HumanoidRootPart.CFrame = SavedPosition
       end
   end,
})

-- === ABA EXTRAS ===
local TabExtras = Window:CreateTab("Extras", 4483362458)

TabExtras:CreateLabel("Use com cuidado - prefira private servers!")

TabExtras:CreateToggle({
    Name = "Anti-AFK / Anti-Kick",
    CurrentValue = false,
    Callback = function(Value)
        AntiAFKEnabled = Value
    end,
})

spawn(function()
    while task.wait(60) do
        if AntiAFKEnabled then
            VU:CaptureController()
            VU:ClickButton2(Vector2.new())
        end
    end
end)

TabExtras:CreateSection("Scripts Externos")

TabExtras:CreateButton({
   Name = "Wall Hop Script",
   Callback = function()
       loadstring(game:HttpGet("https://raw.githubusercontent.com/AhmadV99/Script-Games/main/Wall%20Hop.lua"))()
   end,
})

TabExtras:CreateButton({
   Name = "Simple Fly Script",
   Callback = function()
       loadstring(game:HttpGet("https://raw.githubusercontent.com/BlueRedYellowGreen/Simple-Fly/main/SimpleFly.lua"))() -- Link estável alternativo
   end,
})

TabExtras:CreateButton({
   Name = "Infinite Yield Admin",
   Callback = function()
       loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
   end,
})

Rayfield:Notify({
    Title = "Aviso Importante",
    Content = "Hub para You vs Jon / BEAR Alpha. Use em alt ou private server. Pode ser detectado por anti-cheats (noclip/ESP/highlight). Sem garantia.",
    Duration = 7,
    Image = 4483362458
})

-- === LOOPS ===
RS.RenderStepped:Connect(function()
    if ESPEnabled or ChamsEnabled then
        task.wait(0.03) -- pequeno delay pra menos lag
        updateESP()
    end
end)

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        task.wait(1.2)
        if ESPEnabled or ChamsEnabled then updateESP() end
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = speedValue
        end
    end)
end)

Players.PlayerRemoving:Connect(function()
    task.wait(0.4)
    if ESPEnabled or ChamsEnabled then updateESP() end
end)

Player.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid").WalkSpeed = speedValue
end)    hl.Adornee = character
    hl.FillColor = color
    hl.OutlineColor = color
    hl.FillTransparency = 0.35
    hl.OutlineTransparency = 0.1
    hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    hl.Parent = character
    espHighlights[character] = hl
end

local function updateESP()
    if not ESPEnabled then return end
    ClearESP()  -- Limpa e recria para evitar bugs

    for _, v in pairs(Players:GetPlayers()) do
        if v \~= Player and v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
            local color = (v.Team == Player.Team and Color3.fromRGB(0, 255, 0)) or Color3.fromRGB(255, 0, 0)
            createESP(v.Character, color)
        end
    end
end

-- === ABA PRINCIPAL ===
local TabMain = Window:CreateTab("Principal", 4483362458)

TabMain:CreateSection("Visão 👁️")

TabMain:CreateToggle({
    Name = "ESP (Caixas + Nome/Team)",
    CurrentValue = false,
    Callback = function(Value)
        ESPEnabled = Value
        if not Value then ClearESP() else updateESP() end
    end,
})

TabMain:CreateToggle({
    Name = "Chams (Highlight Corpo)",
    CurrentValue = false,
    Callback = function(Value)
        ChamsEnabled = Value
        -- Pode ser toggle separado ou junto com ESP
        if Value then updateESP() else ClearESP() end
    end,
})

TabMain:CreateToggle({
    Name = "Ghost Mode (Noclip Melhorado)",
    CurrentValue = false,
    Callback = function(Value)
        GhostModeEnabled = Value
        if Value then
            noclipConnection = RunService.Stepped:Connect(function()
                if Player.Character then
                    for _, part in pairs(Player.Character:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.CanCollide = false
                        end
                    end
                end
            end)
        else
            if noclipConnection then noclipConnection:Disconnect() noclipConnection = nil end
        end
    end,
})

TabMain:CreateSection("Visual")

TabMain:CreateToggle({
    Name = "Full Bright + No Fog",
    CurrentValue = false,
    Callback = function(Value)
        FullBrightEnabled = Value
        if Value then
            for _, v in pairs(Lighting:GetChildren()) do
                if v:IsA("PostEffect") then v.Enabled = false end
            end
            Lighting.Brightness = 5
            Lighting.ClockTime = 14
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 999999
            Lighting.Ambient = Color3.fromRGB(200, 200, 200)
        else
            -- Restaura
            Lighting.Brightness = defaultLighting.Brightness or 1
            Lighting.ClockTime = defaultLighting.ClockTime or 14
            Lighting.GlobalShadows = defaultLighting.GlobalShadows \~= false
            Lighting.FogEnd = defaultLighting.FogEnd or 100000
            Lighting.Ambient = defaultLighting.Ambient or Color3.fromRGB(0,0,0)
            for name, data in pairs(defaultLighting) do
                local effect = Lighting:FindFirstChild(name)
                if effect and data.Enabled \~= nil then effect.Enabled = data.Enabled end
            end
        end
    end,
})

TabMain:CreateSection("Movimento")

local speedSlider = TabMain:CreateSlider({
    Name = "WalkSpeed",
    Range = {16, 100},
    Increment = 1,
    Suffix = "Speed",
    CurrentValue = 16,
    Callback = function(Value)
        if Player.Character and Player.Character:FindFirstChild("Humanoid") then
            Player.Character.Humanoid.WalkSpeed = Value
        end
    end,
})

TabMain:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Callback = function(Value)
        InfiniteJumpEnabled = Value
    end,
})

UserInputService.JumpRequest:Connect(function()
    if InfiniteJumpEnabled and Player.Character then
        Player.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
    end
end)

TabMain:CreateSection("Sistema de Teleporte")

TabMain:CreateButton({
   Name = "Salvar Posição",
   Callback = function()
       if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
           SavedPosition = Player.Character.HumanoidRootPart.CFrame
           Rayfield:Notify({Title = "Sucesso", Content = "Posição salva!", Duration = 3})
       end
   end,
})

TabMain:CreateButton({
   Name = "Teleportar para Salvo",
   Callback = function()
       if SavedPosition and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
           Player.Character.HumanoidRootPart.CFrame = SavedPosition
       end
   end,
})

-- === ABA EXTRAS ===
local TabExtras = Window:CreateTab("Extras / Anti-Ban", 4483362458)

TabExtras:CreateToggle({
    Name = "Anti-AFK / Anti-Kick",
    CurrentValue = false,
    Callback = function(Value)
        AntiAFKEnabled = Value
    end,
})

spawn(function()
    while wait(60) do
        if AntiAFKEnabled then
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new())
        end
    end
end)

TabExtras:CreateSection("Scripts Externos (use com cuidado)")

TabExtras:CreateButton({
   Name = "Wall Hop Script",
   Callback = function()
       loadstring(game:HttpGet("https://raw.githubusercontent.com/AhmadV99/Script-Games/main/Wall%20Hop.lua"))()
   end,
})

TabExtras:CreateButton({
   Name = "Fly Script (V3 ou similar)",
   Callback = function()
       loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/Example.lua"))()  -- Exemplo, troque se precisar
   end,
})

TabExtras:CreateButton({
   Name = "Infinite Yield (Admin Commands)",
   Callback = function()
       loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
   end,
})

Rayfield:Notify({
    Title = "Aviso Importante",
    Content = "Use em private server ou alt account. Anti-cheats em BEAR/You vs Jon podem detectar noclip/ESP/fullbright. Sem garantia de não-ban.",
    Duration = 8,
    Image = 4483362458
})

-- === LOOPS ===
RunService.RenderStepped:Connect(function()
    if ESPEnabled or ChamsEnabled then
        updateESP()
    end
end)

Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        wait(1)
        if ESPEnabled or ChamsEnabled then updateESP() end
    end)
end)

Players.PlayerRemoving:Connect(function()
    wait(0.5)
    if ESPEnabled or ChamsEnabled then updateESP() end
end)

-- Atualiza speed se humanoid resetar
Player.CharacterAdded:Connect(function(char)
    char:WaitForChild("Humanoid").WalkSpeed = speedSlider.CurrentValue
end)
