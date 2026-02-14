# Script Universal Aimbot
-- =========================================================
-- LEIROZA | AIMBOT ONLY + FOV INPUT NUMBER
-- =========================================================

-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local LP = Players.LocalPlayer

-- ================= CONFIG =================
local CFG = {
	AIMBOT = false,
	TEAMCHECK = true,
	SHOW_FOV = true,
	FOV = 120
}

-- ================= UI =================
local gui = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
gui.Name = "LeirozaAimbot"
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,360,0,260)
main.Position = UDim2.new(0.5,-180,0.5,-130)
main.BackgroundColor3 = Color3.fromRGB(35,35,35)
main.Active = true
main.Draggable = true
Instance.new("UICorner", main)

-- TOP
local top = Instance.new("Frame", main)
top.Size = UDim2.new(1,0,0,40)
top.BackgroundColor3 = Color3.fromRGB(20,20,20)
Instance.new("UICorner", top)

local title = Instance.new("TextLabel", top)
title.Size = UDim2.new(1,-90,1,0)
title.Position = UDim2.new(0,10,0,0)
title.BackgroundTransparency = 1
title.Text = "LEIROZA | AIMBOT"
title.Font = Enum.Font.Code
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(0,255,140)
title.TextXAlignment = Enum.TextXAlignment.Left

-- MINIMIZE
local btnMin = Instance.new("TextButton", top)
btnMin.Size = UDim2.new(0,30,0,24)
btnMin.Position = UDim2.new(1,-70,0.5,-12)
btnMin.Text = "-"
btnMin.Font = Enum.Font.Code
btnMin.TextSize = 18
btnMin.BackgroundColor3 = Color3.fromRGB(0,120,0)
btnMin.TextColor3 = Color3.new(1,1,1)
btnMin.BorderSizePixel = 0
Instance.new("UICorner", btnMin)

-- CLOSE
local btnClose = Instance.new("TextButton", top)
btnClose.Size = UDim2.new(0,30,0,24)
btnClose.Position = UDim2.new(1,-35,0.5,-12)
btnClose.Text = "X"
btnClose.Font = Enum.Font.Code
btnClose.TextSize = 14
btnClose.BackgroundColor3 = Color3.fromRGB(150,0,0)
btnClose.TextColor3 = Color3.new(1,1,1)
btnClose.BorderSizePixel = 0
Instance.new("UICorner", btnClose)

-- BODY
local body = Instance.new("Frame", main)
body.Position = UDim2.new(0,0,0,40)
body.Size = UDim2.new(1,0,1,-40)
body.BackgroundTransparency = 1

-- TOGGLE
local function toggle(y,text,callback)
	local b = Instance.new("TextButton", body)
	b.Size = UDim2.new(1,-20,0,30)
	b.Position = UDim2.new(0,10,0,y)
	b.Font = Enum.Font.Code
	b.TextSize = 15
	b.TextColor3 = Color3.new(1,1,1)
	b.BackgroundColor3 = Color3.fromRGB(60,60,60)
	b.Text = text..": OFF"
	Instance.new("UICorner", b)

	b.MouseButton1Click:Connect(function()
		callback(b)
	end)
end

-- AIMBOT TOGGLE
toggle(10,"AIMBOT",function(b)
	CFG.AIMBOT = not CFG.AIMBOT
	b.Text = "AIMBOT: "..(CFG.AIMBOT and "ON" or "OFF")
	b.BackgroundColor3 = CFG.AIMBOT and Color3.fromRGB(0,140,0) or Color3.fromRGB(60,60,60)
end)

-- SHOW FOV
toggle(50,"SHOW FOV",function(b)
	CFG.SHOW_FOV = not CFG.SHOW_FOV
	b.Text = "SHOW FOV: "..(CFG.SHOW_FOV and "ON" or "OFF")
	b.BackgroundColor3 = CFG.SHOW_FOV and Color3.fromRGB(0,140,0) or Color3.fromRGB(60,60,60)
end)

-- TEAM CHECK
toggle(90,"TEAM CHECK",function(b)
	CFG.TEAMCHECK = not CFG.TEAMCHECK
	b.Text = "TEAM CHECK: "..(CFG.TEAMCHECK and "ON" or "OFF")
	b.BackgroundColor3 = CFG.TEAMCHECK and Color3.fromRGB(0,140,0) or Color3.fromRGB(60,60,60)
end)

-- FOV INPUT NUMBER
local box = Instance.new("TextBox", body)
box.Size = UDim2.new(1,-20,0,30)
box.Position = UDim2.new(0,10,0,140)
box.PlaceholderText = "FOV (ex: 120)"
box.Text = tostring(CFG.FOV)
box.Font = Enum.Font.Code
box.TextSize = 15
box.TextColor3 = Color3.new(1,1,1)
box.BackgroundColor3 = Color3.fromRGB(50,50,50)
box.BorderSizePixel = 0
Instance.new("UICorner", box)

box.FocusLost:Connect(function()
	local v = tonumber(box.Text)
	if v then
		CFG.FOV = math.clamp(v,10,500)
		box.Text = tostring(CFG.FOV)
	end
end)

-- ================= FOV CIRCLE =================
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 2
fovCircle.Filled = false
fovCircle.Color = Color3.fromRGB(0,255,140)

-- ================= LOOP =================
RunService.RenderStepped:Connect(function()
	fovCircle.Visible = CFG.SHOW_FOV
	fovCircle.Radius = CFG.FOV
	fovCircle.Position = Vector2.new(
		Camera.ViewportSize.X/2,
		Camera.ViewportSize.Y/2
	)

	if CFG.AIMBOT and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
		local closest, dist

		for _,plr in ipairs(Players:GetPlayers()) do
			if plr ~= LP and plr.Character and plr.Character:FindFirstChild("Head") then
				if CFG.TEAMCHECK and plr.Team == LP.Team then continue end

				local pos, vis = Camera:WorldToViewportPoint(plr.Character.Head.Position)
				if vis then
					local d = (Vector2.new(pos.X,pos.Y) - fovCircle.Position).Magnitude
					if d <= CFG.FOV and (not dist or d < dist) then
						dist = d
						closest = plr.Character.Head
					end
				end
			end
		end

		if closest then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Position)
		end
	end
end)

-- ================= MINIMIZE / CLOSE =================
local miniBtn

btnMin.MouseButton1Click:Connect(function()
	if miniBtn then return end
	main.Visible = false

	miniBtn = Instance.new("TextButton", gui)
	miniBtn.Size = UDim2.new(0,120,0,32)
	miniBtn.Position = UDim2.new(0,20,0.5,-16)
	miniBtn.Text = "LEIROZA"
	miniBtn.Font = Enum.Font.Code
	miniBtn.TextSize = 16
	miniBtn.TextColor3 = Color3.fromRGB(0,255,140)
	miniBtn.BackgroundColor3 = Color3.fromRGB(30,30,30)
	miniBtn.Active = true
	miniBtn.Draggable = true
	miniBtn.BorderSizePixel = 0
	Instance.new("UICorner", miniBtn)

	miniBtn.MouseButton1Click:Connect(function()
		main.Visible = true
		miniBtn:Destroy()
		miniBtn = nil
	end)
end)

btnClose.MouseButton1Click:Connect(function()
	fovCircle:Remove()
	gui:Destroy()
end)

-- ================= F8 TOGGLE AIMBOT =================
UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.F8 then
		CFG.AIMBOT = not CFG.AIMBOT

		-- Atualiza botão do aimbot se existir
		for _,v in pairs(body:GetChildren()) do
			if v:IsA("TextButton") and v.Text:find("AIMBOT") then
				v.Text = "AIMBOT: "..(CFG.AIMBOT and "ON" or "OFF")
				v.BackgroundColor3 = CFG.AIMBOT 
					and Color3.fromRGB(0,140,0) 
					or Color3.fromRGB(60,60,60)
			end
		end
	end
end)
