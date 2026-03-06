--[[
    CYBER SCRIPT | AUTO-TYCOON + ALL COLLECTOR + NPC AIMBOT (CORRIGIDO)
    Criador: CyberNoDry
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local PlayerGui = Player:WaitForChild("PlayerGui")
local Camera = workspace.CurrentCamera

-- Variáveis de Controle
local autoBuildEnabled = false
local autoCrateEnabled = false
local npcAimbotEnabled = false
local Holding = false 
local FOV_Radius = 150
local collectorPartPath = workspace.Plots["6"].CollectorParts.Collector
local buttonsFolder = workspace.Plots["6"].Buttons

-- Círculo de FOV (Visual)
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.Filled = false
FOVCircle.Transparency = 0.7
FOVCircle.Visible = false

-- Interface (GUI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CyberHub_Tycoon"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 250, 0, 280)
MainFrame.Position = UDim2.new(0.5, -125, 0.5, -140)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true 
MainFrame.Parent = ScreenGui

local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 2
UIStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
UIStroke.Parent = MainFrame

spawn(function()
    while task.wait() do
        local hue = tick() % 5 / 5
        local color = Color3.fromHSV(hue, 1, 1)
        UIStroke.Color = color
        FOVCircle.Color = color
    end
end)

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.Text = "TYCOON da EvoluEvolução da Idade"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.GothamBold
Title.TextSize = 14
Title.Parent = MainFrame

local function styleButton(btn, pos, text)
    btn.Size = UDim2.new(0.8, 0, 0, 35)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 50, 50)
    btn.Font = Enum.Font.GothamSemibold
    btn.Parent = MainFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
end

local ToggleBtn = Instance.new("TextButton")
styleButton(ToggleBtn, UDim2.new(0.1, 0, 0.20, 0), "AUTO-TYCOON: OFF")

local CrateBtn = Instance.new("TextButton")
styleButton(CrateBtn, UDim2.new(0.1, 0, 0.40, 0), "PEGAR CAIXAS: OFF")

local AimbotBtn = Instance.new("TextButton")
styleButton(AimbotBtn, UDim2.new(0.1, 0, 0.60, 0), "AIMBOT NPC: OFF")

local function createSmallBtn(text, pos)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 25, 0, 25)
    btn.Position = pos
    btn.Text = text
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundTransparency = 1
    btn.Parent = MainFrame
    return btn
end
local MinBtn = createSmallBtn("-", UDim2.new(0.8, 0, 0, 5))
local CloseBtn = createSmallBtn("X", UDim2.new(0.9, 0, 0, 5))

--- LÓGICA ---

local crateNames = {"Wood Money Crate", "Metal Money Crate", "Gold Money Crate", "Diamond Money Crate"}

local function getCrates()
    while autoCrateEnabled do
        for _, obj in pairs(workspace:GetChildren()) do
            if not autoCrateEnabled then break end
            if table.find(crateNames, obj.Name) and obj:IsA("BasePart") then
                HumanoidRootPart.CFrame = obj.CFrame
                task.wait(0.2)
            end
        end
        task.wait(0.5)
    end
end

local function autoBuild()
    while autoBuildEnabled do
        if collectorPartPath then
            firetouchinterest(HumanoidRootPart, collectorPartPath, 0)
            firetouchinterest(HumanoidRootPart, collectorPartPath, 1)
        end
        for _, btn in pairs(buttonsFolder:GetChildren()) do
            if not autoBuildEnabled then break end
            if btn:IsA("BasePart") and btn.Transparency < 1 then
                HumanoidRootPart.CFrame = btn.CFrame
                task.wait(0.3)
            end
        end
        task.wait(0.5)
    end
end

local function GetClosestNPC()
    local target = nil
    local minDist = FOV_Radius
    local npcsFolder = workspace:FindFirstChild("Npcs")
    if npcsFolder then
        for _, npc in pairs(npcsFolder:GetChildren()) do
            local part = npc:FindFirstChild("Torso") or npc:FindFirstChild("HumanoidRootPart")
            if part then
                local pos, onScreen = Camera:WorldToScreenPoint(part.Position)
                if onScreen then
                    local mag = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                    if mag < minDist then
                        target = part
                        minDist = mag
                    end
                end
            end
        end
    end
    return target
end

RunService.RenderStepped:Connect(function()
    if npcAimbotEnabled then
        FOVCircle.Visible = true
        FOVCircle.Radius = FOV_Radius
        FOVCircle.Position = UserInputService:GetMouseLocation()
    else
        FOVCircle.Visible = false
    end

    if npcAimbotEnabled and Holding then
        local target = GetClosestNPC()
        if target then
            Camera.CFrame = Camera.CFrame:Lerp(CFrame.lookAt(Camera.CFrame.Position, target.Position), 0.2)
        end
    end
end)

--- EVENTOS ---

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then Holding = true end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then Holding = false end
end)

ToggleBtn.MouseButton1Click:Connect(function()
    autoBuildEnabled = not autoBuildEnabled
    ToggleBtn.Text = autoBuildEnabled and "AUTO-TYCOON: ON" or "AUTO-TYCOON: OFF"
    ToggleBtn.TextColor3 = autoBuildEnabled and Color3.fromRGB(50, 255, 100) or Color3.fromRGB(255, 50, 50)
    if autoBuildEnabled then task.spawn(autoBuild) end
end)

CrateBtn.MouseButton1Click:Connect(function()
    autoCrateEnabled = not autoCrateEnabled
    CrateBtn.Text = autoCrateEnabled and "PEGAR CAIXAS: ON" or "PEGAR CAIXAS: OFF"
    CrateBtn.TextColor3 = autoCrateEnabled and Color3.fromRGB(50, 255, 100) or Color3.fromRGB(255, 50, 50)
    if autoCrateEnabled then task.spawn(getCrates) end
end)

AimbotBtn.MouseButton1Click:Connect(function()
    npcAimbotEnabled = not npcAimbotEnabled
    AimbotBtn.Text = npcAimbotEnabled and "AIMBOT NPC: ON" or "AIMBOT NPC: OFF"
    AimbotBtn.TextColor3 = npcAimbotEnabled and Color3.fromRGB(50, 255, 100) or Color3.fromRGB(255, 50, 50)
end)

-- LÓGICA DE MINIMIZAR CORRIGIDA
local minimized = false
MinBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    
    -- Esconde ou mostra os botões para não bugar
    ToggleBtn.Visible = not minimized
    CrateBtn.Visible = not minimized
    AimbotBtn.Visible = not minimized
    
    -- Ajusta o tamanho do frame
    if minimized then
        MainFrame:TweenSize(UDim2.new(0, 250, 0, 35), "Out", "Quad", 0.3, true)
        MinBtn.Text = "+"
    else
        MainFrame:TweenSize(UDim2.new(0, 250, 0, 280), "Out", "Quad", 0.3, true)
        MinBtn.Text = "-"
    end
end)

CloseBtn.MouseButton1Click:Connect(function()
    FOVCircle:Remove()
    ScreenGui:Destroy()
end)
