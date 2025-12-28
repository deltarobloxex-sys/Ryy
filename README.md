-- ========================================
-- DELTA MENU COMPLETO + AIMBOT + ESP
-- ========================================

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- =========================
-- CONFIGURAÇÕES
-- =========================
local Config = {
	Aimbot = false,
	ESP = false,
	FOV = false,
	TeamCheck = true,
	Smooth = 0.15,
	MaxDistance = 1000,
	FOVRadius = 120
}

-- =========================
-- FUNÇÕES
-- =========================
local function IsEnemy(plr)
	if not Config.TeamCheck then return true end
	if not plr.Team or not LocalPlayer.Team then return true end
	if plr.Neutral or LocalPlayer.Neutral then return true end
	return plr.Team ~= LocalPlayer.Team
end

local function GetClosest()
	if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("Head") then return end
	local closest, dist = nil, Config.MaxDistance
	for _, plr in pairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character and IsEnemy(plr) then
			local h = plr.Character:FindFirstChild("Head")
			if h then
				local d = (h.Position - LocalPlayer.Character.Head.Position).Magnitude
				if d < dist then
					dist = d
					closest = h
				end
			end
		end
	end
	return closest
end

-- =========================
-- GUI
-- =========================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "DeltaMenu"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

-- BOTÃO FLUTUANTE
local FloatBtn = Instance.new("ImageButton", ScreenGui)
FloatBtn.Size = UDim2.new(0,60,0,60)
FloatBtn.Position = UDim2.new(0,20,0.5,-30)
FloatBtn.Image = "rbxassetid://7072718367"
FloatBtn.BackgroundTransparency = 1
Instance.new("UICorner", FloatBtn).CornerRadius = UDim.new(1,0)

-- PAINEL
local Panel = Instance.new("Frame", ScreenGui)
Panel.Size = UDim2.new(0,320,0,380)
Panel.Position = UDim2.new(0.05,0,0.05,0)
Panel.BackgroundColor3 = Color3.fromRGB(18,18,25)
Panel.Visible = true
Instance.new("UICorner", Panel).CornerRadius = UDim.new(0,18)

-- TÍTULO
local Title = Instance.new("TextLabel", Panel)
Title.Size = UDim2.new(1,0,0,50)
Title.BackgroundTransparency = 1
Title.Text = "⚡ DELTA MENU"
Title.Font = Enum.Font.GothamBlack
Title.TextSize = 20
Title.TextColor3 = Color3.fromRGB(0,170,255)

-- FUNÇÃO BOTÃO
local function NewButton(text,y)
	local b = Instance.new("TextButton", Panel)
	b.Size = UDim2.new(0.85,0,0,40)
	b.Position = UDim2.new(0.075,0,y,0)
	b.BackgroundColor3 = Color3.fromRGB(35,35,50)
	b.Text = text
	b.Font = Enum.Font.GothamBold
	b.TextSize = 13
	b.TextColor3 = Color3.new(1,1,1)
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,10)
	return b
end

local AimbotBtn = NewButton("🎯 AIMBOT: OFF",0.20)
local ESPBtn    = NewButton("👁️ ESP: OFF",0.32)
local FOVBtn    = NewButton("⭕ FOV: OFF",0.44)
local TeamBtn   = NewButton("🛡️ TEAM CHECK: ON",0.56)
local CloseBtn  = NewButton("❌ FECHAR",0.72)

local function Toggle(btn,on)
	btn.BackgroundColor3 = on and Color3.fromRGB(0,170,255) or Color3.fromRGB(35,35,50)
end

-- =========================
-- CONEXÕES BOTÕES
-- =========================
AimbotBtn.MouseButton1Click:Connect(function()
	Config.Aimbot = not Config.Aimbot
	AimbotBtn.Text = "🎯 AIMBOT: "..(Config.Aimbot and "ON" or "OFF")
	Toggle(AimbotBtn,Config.Aimbot)
end)

ESPBtn.MouseButton1Click:Connect(function()
	Config.ESP = not Config.ESP
	ESPBtn.Text = "👁️ ESP: "..(Config.ESP and "ON" or "OFF")
	Toggle(ESPBtn,Config.ESP)
end)

FOVBtn.MouseButton1Click:Connect(function()
	Config.FOV = not Config.FOV
	FOVBtn.Text = "⭕ FOV: "..(Config.FOV and "ON" or "OFF")
	Toggle(FOVBtn,Config.FOV)
end)

TeamBtn.MouseButton1Click:Connect(function()
	Config.TeamCheck = not Config.TeamCheck
	TeamBtn.Text = "🛡️ TEAM CHECK: "..(Config.TeamCheck and "ON" or "OFF")
	Toggle(TeamBtn,Config.TeamCheck)
end)

-- =========================
-- ABRIR / FECHAR ANIMADO
-- =========================
local menuOpen = true
Panel.Visible = true
local tweenInfo = TweenInfo.new(0.3,Enum.EasingStyle.Quad,Enum.EasingDirection.Out)

FloatBtn.MouseButton1Click:Connect(function()
	menuOpen = not menuOpen
	TweenService:Create(Panel,tweenInfo,{Position=menuOpen and UDim2.new(0.05,0,0.05,0) or UDim2.new(-0.4,0,0.05,0)}):Play()
end)

CloseBtn.MouseButton1Click:Connect(function()
	menuOpen = false
	TweenService:Create(Panel,tweenInfo,{Position=UDim2.new(-0.4,0,0.05,0)}):Play()
end)

-- =========================
-- DRAG MENU (PC + MOBILE)
-- =========================
local dragging = false
local dragStart
local startPos

local function updateDrag(input)
	local delta = input.Position - dragStart
	Panel.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)
end

Title.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
	or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = Panel.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and
	(input.UserInputType == Enum.UserInputType.MouseMovement
	or input.UserInputType == Enum.UserInputType.Touch) then
		updateDrag(input)
	end
end)

-- =========================
-- FOV CIRCLE
-- =========================
local FOVCircle = Instance.new("Frame", ScreenGui)
FOVCircle.Size = UDim2.new(0,Config.FOVRadius*2,0,Config.FOVRadius*2)
FOVCircle.Position = UDim2.new(0.5,-Config.FOVRadius,0.5,-Config.FOVRadius)
FOVCircle.BackgroundTransparency = 1
Instance.new("UICorner",FOVCircle).CornerRadius = UDim.new(1,0)
local Stroke = Instance.new("UIStroke",FOVCircle)
Stroke.Color = Color3.fromRGB(0,170,255)
Stroke.Thickness = 2

-- =========================
-- ESP BOX + LINE
-- =========================
local ESPObjects = {}

local function GetESP(plr)
	if ESPObjects[plr] then return ESPObjects[plr] end
	local box = Instance.new("Frame", ScreenGui)
	box.BorderSizePixel = 2
	box.BackgroundTransparency = 1
	box.BorderColor3 = Color3.fromRGB(0,255,100)
	local line = Drawing.new("Line")
	line.Thickness = 1
	line.Color = Color3.fromRGB(0,255,100)
	ESPObjects[plr] = {Box=box, Line=line}
	return ESPObjects[plr]
end

-- =========================
-- LOOP PRINCIPAL
-- =========================
RunService.RenderStepped:Connect(function()
	FOVCircle.Visible = Config.FOV

	if Config.ESP then
		for _,plr in pairs(Players:GetPlayers()) do
			if plr~=LocalPlayer and plr.Character and IsEnemy(plr) then
				local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
				if hrp then
					local esp = GetESP(plr)
					local pos,on = Camera:WorldToViewportPoint(hrp.Position)
					if on then
						esp.Box.Visible = true
						esp.Box.Size = UDim2.new(0,40,0,60)
						esp.Box.Position = UDim2.new(0,pos.X-20,0,pos.Y-30)
						esp.Line.Visible = true
						esp.Line.From = Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y)
						esp.Line.To = Vector2.new(pos.X,pos.Y)
					else
						esp.Box.Visible = false
						esp.Line.Visible = false
					end
				end
			end
		end
	end

	if Config.Aimbot then
		local head = GetClosest()
		if head then
			Camera.CFrame = Camera.CFrame:Lerp(
				CFrame.new(Camera.CFrame.Position,head.Position),
				Config.Smooth
			)
		end
	end
end)-- =========================
-- BOTÃO FLUTUANTE ⚡ ABRIR/FECHAR
-- =========================
local menuOpen = true -- estado inicial do menu
Panel.Visible = true -- menu começa aberto
local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)

-- Criando o botão ⚡ (se ainda não existir)
local OpenBtn = Instance.new("TextButton")
OpenBtn.Size = UDim2.new(0,50,0,50)
OpenBtn.Position = UDim2.new(0,20,0.5,-25)
OpenBtn.BackgroundColor3 = Color3.fromRGB(0,170,255)
OpenBtn.Text = "⚡"
OpenBtn.TextScaled = true
OpenBtn.TextColor3 = Color3.new(1,1,1)
OpenBtn.ZIndex = 10
OpenBtn.Active = true
OpenBtn.Parent = ScreenGui
Instance.new("UICorner", OpenBtn).CornerRadius = UDim.new(1,0)

-- Função abrir/fechar menu
local function ToggleMenu()
    menuOpen = not menuOpen
    local targetPos = menuOpen and UDim2.new(0.05,0,0.05,0) or UDim2.new(-0.4,0,0.05,0)
    TweenService:Create(Panel, tweenInfo, {Position = targetPos}):Play()
end

-- Conectar clique
OpenBtn.MouseButton1Click:Connect(ToggleMenu)
