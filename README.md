--[[ Arsenal Hack sem Auto Kill e Fly ]]--

-- Serviços e inicialização
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local AimbotEnabled = false
local ESPEnabled = false
local TeamCheck = true

-- GUI principal
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "ArsenalHackUI"
gui.ResetOnSpawn = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 260, 0, 300)
frame.Position = UDim2.new(0.05, 0, 0.1, 0)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0

local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 12)

local function createToggle(text, callback)
    local btn = Instance.new("TextButton", frame)
    btn.Size = UDim2.new(0.9, 0, 0, 30)
    btn.Position = UDim2.new(0.05, 0, 0, (#frame:GetChildren() - 1) * 35)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Text = text .. ": OFF"
    btn.AutoButtonColor = false

    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.Text = text .. ": " .. (state and "ON" or "OFF")
        callback(state)
    end)

    local corner = Instance.new("UICorner", btn)
    corner.CornerRadius = UDim.new(0, 6)
end

-- Toggles
createToggle("Aimbot", function(val) AimbotEnabled = val end)
createToggle("ESP", function(val) ESPEnabled = val end)
createToggle("Team Check", function(val) TeamCheck = val end)

-- Botão flutuante fora da interface para minimizar
local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Parent = gui
minimizeBtn.Size = UDim2.new(0, 80, 0, 30)
minimizeBtn.Position = UDim2.new(0, 10, 0, 10)
minimizeBtn.Text = "Menu"
minimizeBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.Font = Enum.Font.Gotham
minimizeBtn.TextSize = 14
minimizeBtn.AutoButtonColor = true

local minimizeCorner = Instance.new("UICorner", minimizeBtn)
minimizeCorner.CornerRadius = UDim.new(0, 8)

minimizeBtn.MouseButton1Click:Connect(function()
    frame.Visible = not frame.Visible
end)

-- Funções de combate
local function getClosestEnemy()
    local closest, dist = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            if TeamCheck and p.Team == LocalPlayer.Team then continue end
            local pos, visible = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if visible then
                local d = (Vector2.new(pos.X, pos.Y) - Camera.ViewportSize / 2).Magnitude
                if d < dist then
                    dist = d
                    closest = p
                end
            end
        end
    end
    return closest
end

RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local target = getClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)

-- ESP
local function createESP(plr)
    local text = Drawing.new("Text")
    text.Center = true
    text.Size = 14
    text.Outline = true
    text.Color = Color3.fromRGB(255, 0, 0)

    RunService.RenderStepped:Connect(function()
        if plr.Character and plr.Character:FindFirstChild("Head") and ESPEnabled then
            local pos, onscreen = Camera:WorldToViewportPoint(plr.Character.Head.Position)
            text.Position = Vector2.new(pos.X, pos.Y - 30)
            text.Text = plr.Name
            text.Visible = onscreen
        else
            text.Visible = false
        end
    end)
end

for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then createESP(p) end
end

Players.PlayerAdded:Connect(function(p)
    if p ~= LocalPlayer then createESP(p) end
end)
