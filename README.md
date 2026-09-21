local Players = game:GetService("Players")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")

local player = Players.LocalPlayer

-- === Состояния и Настройки ===
local isRunning = false
local scriptActive = true
local isCollapsed = false
local offsetX = 0
local offsetY = -60
local step = 10

-- === ScreenGui ===
local ScreenGui = Instance.new("ScreenGui")
local success = pcall(function() ScreenGui.Parent = gethui and gethui() or CoreGui end)
if not success then ScreenGui.Parent = player:WaitForChild("PlayerGui") end

-- Главный контейнер
local MainFrame = Instance.new("Frame")
MainFrame.Name = "AutoClickerFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(24, 24, 28)
MainFrame.Position = UDim2.new(0.5, -110, 0.5, -170)
MainFrame.Size = UDim2.new(0, 220, 0, 360)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.ClipsDescendants = true

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Color3.fromRGB(50, 50, 60)
MainStroke.Thickness = 1
MainStroke.Parent = MainFrame

-- Шапка (Header)
local Header = Instance.new("Frame", MainFrame)
Header.Size = UDim2.new(1, 0, 0, 40)
Header.BackgroundColor3 = Color3.fromRGB(35, 35, 42)

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 12)
HeaderCorner.Parent = Header

local Title = Instance.new("TextLabel", Header)
Title.BackgroundTransparency = 1
Title.Position = UDim2.new(0, 12, 0, 0)
Title.Size = UDim2.new(0.6, 0, 1, 0)
Title.Font = Enum.Font.GothamBold
Title.Text = "Auto Clicker"
Title.TextColor3 = Color3.fromRGB(240, 240, 240)
Title.TextSize = 14
Title.TextXAlignment = Enum.TextXAlignment.Left

-- Кнопка сворачивания
local MinimizeBtn = Instance.new("TextButton", Header)
MinimizeBtn.BackgroundTransparency = 1
MinimizeBtn.Position = UDim2.new(1, -35, 0, 5)
MinimizeBtn.Size = UDim2.new(0, 30, 0, 30)
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.Text = "—"
MinimizeBtn.TextColor3 = Color3.fromRGB(180, 180, 190)
MinimizeBtn.TextSize = 14

-- Контейнер для основного содержимого (скрывается при сворачивании)
local ContentFrame = Instance.new("Frame", MainFrame)
ContentFrame.BackgroundTransparency = 1
ContentFrame.Position = UDim2.new(0, 0, 0, 40)
ContentFrame.Size = UDim2.new(1, 0, 1, -40)

-- Статус
local StatusLabel = Instance.new("TextLabel", ContentFrame)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Position = UDim2.new(0, 0, 0, 8)
StatusLabel.Size = UDim2.new(1, 0, 0, 18)
StatusLabel.Font = Enum.Font.GothamMedium
StatusLabel.Text = "• Статус: Остановлен"
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatusLabel.TextSize = 12

-- Точка-прицел
local Pointer = Instance.new("Frame", ScreenGui)
Pointer.Size = UDim2.new(0, 8, 0, 8)
Pointer.AnchorPoint = Vector2.new(0.5, 0.5)
Pointer.Position = UDim2.new(0.5, offsetX, 0.5, offsetY)
Pointer.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
Pointer.ZIndex = 100

local PointerCorner = Instance.new("UICorner")
PointerCorner.CornerRadius = UDim.new(1, 0)
PointerCorner.Parent = Pointer

local PointerStroke = Instance.new("UIStroke", Pointer)
PointerStroke.Color = Color3.fromRGB(0, 0, 0)
PointerStroke.Thickness = 1.5

local function updatePointer()
    TweenService:Create(Pointer, TweenInfo.new(0.08), {Position = UDim2.new(0.5, offsetX, 0.5, offsetY)}):Play()
end

-- Вспомогательная функция создания стильных кнопок
local function createButton(parent, text, pos, size, color)
    local btn = Instance.new("TextButton")
    btn.Parent = parent
    btn.BackgroundColor3 = color
    btn.Position = pos
    btn.Size = size
    btn.Font = Enum.Font.GothamBold
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 12

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 8)
    corner.Parent = btn

    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = Color3.new(math.min(color.R*1.25, 1), math.min(color.G*1.25, 1), math.min(color.B*1.25, 1))}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3 = color}):Play()
    end)
    return btn
end

-- Кнопки СТАРТ / СТОП
local StartBtn = createButton(ContentFrame, "СТАРТ", UDim2.new(0.08, 0, 0, 32), UDim2.new(0.84, 0, 0, 32), Color3.fromRGB(46, 125, 50))
local StopBtn = createButton(ContentFrame, "СТОП", UDim2.new(0.08, 0, 0, 70), UDim2.new(0.84, 0, 0, 32), Color3.fromRGB(198, 40, 40))

-- Секция настройки точки
local DPadLabel = Instance.new("TextLabel", ContentFrame)
DPadLabel.BackgroundTransparency = 1
DPadLabel.Position = UDim2.new(0, 0, 0, 112)
DPadLabel.Size = UDim2.new(1, 0, 0, 18)
DPadLabel.Font = Enum.Font.GothamMedium
DPadLabel.Text = "Позиция клика"
DPadLabel.TextColor3 = Color3.fromRGB(150, 150, 160)
DPadLabel.TextSize = 11

local dPadColor = Color3.fromRGB(40, 40, 48)
local UpBtn = createButton(ContentFrame, "▲", UDim2.new(0.5, -14, 0, 134), UDim2.new(0, 28, 0, 28), dPadColor)
local DownBtn = createButton(ContentFrame, "▼", UDim2.new(0.5, -14, 0, 194), UDim2.new(0, 28, 0, 28), dPadColor)
local LeftBtn = createButton(ContentFrame, "◄", UDim2.new(0.5, -46, 0, 164), UDim2.new(0, 28, 0, 28), dPadColor)
local RightBtn = createButton(ContentFrame, "►", UDim2.new(0.5, 18, 0, 164), UDim2.new(0, 28, 0, 28), dPadColor)

local ExitBtn = createButton(ContentFrame, "ВЫКЛЮЧИТЬ", UDim2.new(0.08, 0, 0, 236), UDim2.new(0.84, 0, 0, 32), Color3.fromRGB(60, 60, 70))

-- Подпись "by:vaim"
local AuthorLabel = Instance.new("TextLabel", ContentFrame)
AuthorLabel.BackgroundTransparency = 1
AuthorLabel.Position = UDim2.new(0, 0, 1, -22)
AuthorLabel.Size = UDim2.new(1, 0, 0, 18)
AuthorLabel.Font = Enum.Font.Gotham
AuthorLabel.Text = "by:vaim"
AuthorLabel.TextColor3 = Color3.fromRGB(100, 100, 115)
AuthorLabel.TextSize = 11

-- === Обработчики событий крестовины ===
UpBtn.MouseButton1Click:Connect(function() offsetY = offsetY - step; updatePointer() end)
DownBtn.MouseButton1Click:Connect(function() offsetY = offsetY + step; updatePointer() end)
LeftBtn.MouseButton1Click:Connect(function() offsetX = offsetX - step; updatePointer() end)
RightBtn.MouseButton1Click:Connect(function() offsetX = offsetX + step; updatePointer() end)

-- === Сворачивание / Разворачивание ===
MinimizeBtn.MouseButton1Click:Connect(function()
    isCollapsed = not isCollapsed
    if isCollapsed then
        ContentFrame.Visible = false
        MinimizeBtn.Text = "□"
        TweenService:Create(MainFrame, TweenInfo.new(0.2), {Size = UDim2.new(0, 220, 0, 40)}):Play()
    else
        TweenService:Create(MainFrame, TweenInfo.new(0.2), {Size = UDim2.new(0, 220, 0, 360)}):Play()
        task.wait(0.2)
        ContentFrame.Visible = true
        MinimizeBtn.Text = "—"
    end
end)

-- === Логика кликов и ресета ===
local function clickTarget()
    local camera = Workspace.CurrentCamera
    local viewport = camera.ViewportSize
    local targetX = (viewport.X / 2) + offsetX
    local targetY = (viewport.Y / 2) + offsetY

    VirtualInputManager:SendMouseButtonEvent(targetX, targetY, 0, true, game, 1)
    task.wait(0.05)
    VirtualInputManager:SendMouseButtonEvent(targetX, targetY, 0, false, game, 1)
end

task.spawn(function()
    while scriptActive do
        if isRunning then
            local character = player.Character
            if character then
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid and humanoid.Health > 0 then
                    humanoid.Health = 0
                    task.wait(0.3)
                    clickTarget()
                    player.CharacterAdded:Wait()
                    task.wait(0.2)
                end
            end
        end
        task.wait(0.1)
    end
end)

-- === Управление состоянием ===
StartBtn.MouseButton1Click:Connect(function()
    isRunning = true
    StatusLabel.Text = "• Статус: Работает"
    StatusLabel.TextColor3 = Color3.fromRGB(76, 175, 80)
    Pointer.BackgroundColor3 = Color3.fromRGB(76, 217, 100)
end)

StopBtn.MouseButton1Click:Connect(function()
    isRunning = false
    StatusLabel.Text = "• Статус: Остановлен"
    StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
    Pointer.BackgroundColor3 = Color3.fromRGB(255, 75, 75)
end)

ExitBtn.MouseButton1Click:Connect(function()
    scriptActive = false
    isRunning = false
    ScreenGui:Destroy()
end)
