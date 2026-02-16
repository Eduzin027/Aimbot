-- Eduardo Universal PvP Script | Rayfield UI | FOV Aimbot + ESP | 2026
-- Toggle Aim: X (ou pelo menu)
-- Funciona na maioria dos jogos Roblox (FPS, Arsenal, CBRO, etc.)

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Eduardo Universal Hub",
   LoadingTitle = "Carregando Eduardo Hub",
   LoadingSubtitle = "Universal Aimbot + ESP",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "EduardoUniversal",
      FileName = "config"
   },
   KeySystem = false
})

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")

local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

getgenv().EduardoUniversal = {
    AimEnabled = false,
    FOVRadius = 150,
    ShowFOV = true,
    TeamCheck = true,
    VisibilityCheck = true,  -- Só mira se visível (wall check)
    AliveCheck = true,
    AimPart = "Head",        -- Head, UpperTorso, HumanoidRootPart, etc.
    Smoothness = 0.08,       -- Quanto menor, mais suave (0.01 = instant)
    Prediction = 0.135,      -- Ajuste pro seu ping (0.1-0.2 bom pra BR)
    ESPEnabled = false,
    ESPTeamCheck = true,
    ESPDistance = 4000,
    ESPTracer = true         -- Linha do seu pé até o inimigo
}

-- FOV Circle
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.NumSides = 100
FOVCircle.Radius = getgenv().EduardoUniversal.FOVRadius
FOVCircle.Color = Color3.fromRGB(255, 50, 50)
FOVCircle.Transparency = 0.9
FOVCircle.Filled = false
FOVCircle.Visible = getgenv().EduardoUniversal.ShowFOV

-- Mouse Position
local MousePos = Vector2.new(0, 0)

-- ESP Table + Tracers
local ESPObjects = {}

local function AddESP(plr)
    if plr == LocalPlayer then return end
    
    local Box = Drawing.new("Square")
    Box.Thickness = 2
    Box.Color = Color3.fromRGB(255, 0, 0)
    Box.Transparency = 1
    Box.Filled = false
    Box.Visible = false
    
    local Tracer = Drawing.new("Line")
    Tracer.Thickness = 1.5
    Tracer.Color = Color3.fromRGB(255, 0, 0)
    Tracer.Transparency = 1
    Tracer.Visible = false
    
    local Name = Drawing.new("Text")
    Name.Size = 14
    Name.Center = true
    Name.Outline = true
    Name.Color = Color3.fromRGB(255, 255, 255)
    Name.Visible = false
    
    local Distance = Drawing.new("Text")
    Distance.Size = 12
    Distance.Center = true
    Distance.Outline = true
    Distance.Color = Color3.fromRGB(200, 200, 200)
    Distance.Visible = false
    
    ESPObjects[plr] = {Box = Box, Tracer = Tracer, Name = Name, Distance = Distance}
end

local function RemoveESP(plr)
    if ESPObjects[plr] then
        for _, obj in pairs(ESPObjects[plr]) do obj:Remove() end
        ESPObjects[plr] = nil
    end
end

local function UpdateESP()
    for plr, esp in pairs(ESPObjects) do
        if plr.Character and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 
           and plr.Character:FindFirstChild("HumanoidRootPart") then
            
            if getgenv().EduardoUniversal.ESPTeamCheck and plr.Team == LocalPlayer.Team then
                esp.Box.Visible = false
                esp.Tracer.Visible = false
                esp.Name.Visible = false
                esp.Distance.Visible = false
                continue
            end
            
            local root = plr.Character.HumanoidRootPart
            local head = plr.Character:FindFirstChild("Head") or root
            local dist = (LocalPlayer.Character.HumanoidRootPart.Position - root.Position).Magnitude
            
            if dist > getgenv().EduardoUniversal.ESPDistance then
                esp.Box.Visible = false
                esp.Tracer.Visible = false
                esp.Name.Visible = false
                esp.Distance.Visible = false
                continue
            end
            
            local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local size = (Camera:WorldToViewportPoint(head.Position - Vector3.new(0, 3, 0)) - Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 3, 0))).Magnitude
                esp.Box.Size = Vector2.new(size * 1.5, size * 3)
                esp.Box.Position = Vector2.new(screenPos.X - esp.Box.Size.X / 2, screenPos.Y - esp.Box.Size.Y / 2)
                esp.Box.Visible = true
                
                esp.Name.Text = plr.Name
                esp.Name.Position = Vector2.new(screenPos.X, screenPos.Y - esp.Box.Size.Y / 2 - 16)
                esp.Name.Visible = true
                
                esp.Distance.Text = math.floor(dist) .. " studs"
                esp.Distance.Position = Vector2.new(screenPos.X, screenPos.Y + esp.Box.Size.Y / 2 + 4)
                esp.Distance.Visible = true
                
                if getgenv().EduardoUniversal.ESPTracer then
                    esp.Tracer.From = Vector2.new(Mouse.X, Mouse.Y + 36)
                    esp.Tracer.To = Vector2.new(screenPos.X, screenPos.Y)
                    esp.Tracer.Visible = true
                else
                    esp.Tracer.Visible = false
                end
            else
                esp.Box.Visible = false
                esp.Tracer.Visible = false
                esp.Name.Visible = false
                esp.Distance.Visible = false
            end
        else
            esp.Box.Visible = false
            esp.Tracer.Visible = false
            esp.Name.Visible = false
            esp.Distance.Visible = false
        end
    end
end

-- Get Closest in FOV (com checks)
local function GetClosest()
    local closest, shortest = nil, math.huge
    
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild(getgenv().EduardoUniversal.AimPart) 
           and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            
            if getgenv().EduardoUniversal.TeamCheck and plr.Team == LocalPlayer.Team then continue end
            if getgenv().EduardoUniversal.AliveCheck and plr.Character.Humanoid.Health <= 0 then continue end
            
            local part = plr.Character[getgenv().EduardoUniversal.AimPart]
            local screen, onScreen = Camera:WorldToViewportPoint(part.Position)
            
            if onScreen then
                local dist = (Vector2.new(screen.X, screen.Y) - Vector2.new(Mouse.X, Mouse.Y + 36)).Magnitude
                if dist < getgenv().EduardoUniversal.FOVRadius and dist < shortest then
                    -- Visibility Check (raycast simples)
                    if getgenv().EduardoUniversal.VisibilityCheck then
                        local ray = Ray.new(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position).Unit * 5000)
                        local hit, pos = Workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character})
                        if hit and hit:IsDescendantOf(plr.Character) then
                            closest = plr
                            shortest = dist
                        end
                    else
                        closest = plr
                        shortest = dist
                    end
                end
            end
        end
    end
    return closest
end

-- Aimbot Loop
RunService.RenderStepped:Connect(function()
    MousePos = Vector2.new(Mouse.X, Mouse.Y + 36)  -- Ajuste pro mobile/PC
    FOVCircle.Position = MousePos
    FOVCircle.Radius = getgenv().EduardoUniversal.FOVRadius
    FOVCircle.Visible = getgenv().EduardoUniversal.ShowFOV and getgenv().EduardoUniversal.AimEnabled
    
    if getgenv().EduardoUniversal.AimEnabled then
        local target = GetClosest()
        if target and target.Character then
            local aimPos = target.Character[getgenv().EduardoUniversal.AimPart].Position + (target.Character[getgenv().EduardoUniversal.AimPart].Velocity * getgenv().EduardoUniversal.Prediction)
            local tweenInfo = TweenInfo.new(getgenv().EduardoUniversal.Smoothness, Enum.EasingStyle.Linear)
            TweenService:Create(Camera, tweenInfo, {CFrame = CFrame.lookAt(Camera.CFrame.Position, aimPos)}):Play()
        end
    end
    
    if getgenv().EduardoUniversal.ESPEnabled then
        UpdateESP()
    end
end)

-- Toggle com X
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.X then
        getgenv().EduardoUniversal.AimEnabled = not getgenv().EduardoUniversal.AimEnabled
        Rayfield:Notify({
            Title = "Eduardo Universal",
            Content = "Aimbot: " .. (getgenv().EduardoUniversal.AimEnabled and "ATIVADO" or "DESATIVADO"),
            Duration = 2
        })
    end
end)

-- Adicionar ESP pros players
for _, plr in pairs(Players:GetPlayers()) do
    AddESP(plr)
end
Players.PlayerAdded:Connect(AddESP)
Players.PlayerRemoving:Connect(RemoveESP)

-- GUI Tabs
local AimTab = Window:CreateTab("Aimbot", 4483362458)
AimTab:CreateSection("FOV Aimbot (Gruda no player)")

AimTab:CreateToggle({
    Name = "Aimbot Ativado (X toggle)",
    CurrentValue = false,
    Callback = function(v) getgenv().EduardoUniversal.AimEnabled = v end
})

AimTab:CreateToggle({
    Name = "Team Check",
    CurrentValue = true,
    Callback = function(v) getgenv().EduardoUniversal.TeamCheck = v end
})

AimTab:CreateToggle({
    Name = "Visibility Check (Anti-Wall)",
    CurrentValue = true,
    Callback = function(v) getgenv().EduardoUniversal.VisibilityCheck = v end
})

AimTab:CreateToggle({
    Name = "Mostrar FOV Circle",
    CurrentValue = true,
    Callback = function(v) getgenv().EduardoUniversal.ShowFOV = v end
})

AimTab:CreateSlider({
    Name = "FOV Radius",
    Range = {50, 500},
    Increment = 5,
    CurrentValue = 150,
    Callback = function(v) getgenv().EduardoUniversal.FOVRadius = v end
})

AimTab:CreateSlider({
    Name = "Suavidade",
    Range = {0.01, 0.3},
    Increment = 0.01,
    CurrentValue = 0.08,
    Callback = function(v) getgenv().EduardoUniversal.Smoothness = v end
})

AimTab:CreateSlider({
    Name = "Predição",
    Range = {0, 0.3},
    Increment = 0.005,
    CurrentValue = 0.135,
    Callback = function(v) getgenv().EduardoUniversal.Prediction = v end
})

AimTab:CreateDropdown({
    Name = "Aim Part",
    Options = {"Head", "UpperTorso", "HumanoidRootPart", "LowerTorso"},
    CurrentOption = "Head",
    Callback = function(opt) getgenv().EduardoUniversal.AimPart = opt end
})

local ESPTab = Window:CreateTab("ESP", 4483362458)
ESPTab:CreateSection("Visuals (Box + Tracer + Info)")

ESPTab:CreateToggle({
    Name = "ESP Ativado",
    CurrentValue = false,
    Callback = function(v) getgenv().EduardoUniversal.ESPEnabled = v end
})

ESPTab:CreateToggle({
    Name = "Team Check ESP",
    CurrentValue = true,
    Callback = function(v) getgenv().EduardoUniversal.ESPTeamCheck = v end
})

ESPTab:CreateToggle({
    Name = "Tracer Lines",
    CurrentValue = true,
    Callback = function(v) getgenv().EduardoUniversal.ESPTracer = v end
})

ESPTab:CreateSlider({
    Name = "Distância Máx",
    Range = {1000, 10000},
    Increment = 500,
    CurrentValue = 4000,
    Callback = function(v) getgenv().EduardoUniversal.ESPDistance = v end
})

print("✅ Eduardo Universal Hub carregado! | Funciona na maioria dos jogos | Team Check ON")
Rayfield:LoadConfiguration()
