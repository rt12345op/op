# local Players=game:GetService("Players")
local RunService=game:GetService("RunService")
local UserInputService=game:GetService("UserInputService")

local player=Players.LocalPlayer
local camera=workspace.CurrentCamera

local gui=Instance.new("ScreenGui")
gui.Name="TestUI"
gui.ResetOnSpawn=false
gui.Parent=player:WaitForChild("PlayerGui")

local frame=Instance.new("Frame")
frame.Size=UDim2.new(0,320,0,200)
frame.Position=UDim2.new(0.5,-160,0.5,-100)
frame.BackgroundColor3=Color3.fromRGB(0,0,0)
frame.BackgroundTransparency=0.2
frame.Active=true
frame.Draggable=true
frame.Parent=gui

local corner=Instance.new("UICorner",frame)
corner.CornerRadius=UDim.new(0,12)

local close=Instance.new("TextButton",frame)
close.Size=UDim2.new(0,24,0,24)
close.Position=UDim2.new(0,6,0,6)
close.Text="X"
close.TextColor3=Color3.fromRGB(255,80,80)
close.BackgroundTransparency=1
close.Font=Enum.Font.GothamBold
close.TextSize=18

local minimize=Instance.new("TextButton",frame)
minimize.Size=UDim2.new(0,24,0,24)
minimize.Position=UDim2.new(0,34,0,6)
minimize.Text="−"
minimize.TextColor3=Color3.fromRGB(255,255,255)
minimize.BackgroundTransparency=1
minimize.Font=Enum.Font.GothamBold
minimize.TextSize=18

local miniIcon=Instance.new("TextButton",gui)
miniIcon.Size=UDim2.new(0,40,0,40)
miniIcon.Position=UDim2.new(0,10,0,50)
miniIcon.BackgroundColor3=Color3.fromRGB(255,255,255)
miniIcon.BackgroundTransparency=0.2
miniIcon.Text=""
miniIcon.Visible=false

local miniCorner=Instance.new("UICorner",miniIcon)
miniCorner.CornerRadius=UDim.new(0,8)

local aimToggle=Instance.new("TextButton",frame)
aimToggle.Size=UDim2.new(0,120,0,30)
aimToggle.Position=UDim2.new(0,20,0,50)
aimToggle.Text="自瞄 NPC: OFF"
aimToggle.BackgroundColor3=Color3.fromRGB(40,40,40)
aimToggle.TextColor3=Color3.new(1,1,1)
aimToggle.Parent=frame

Instance.new("UICorner",aimToggle).CornerRadius=UDim.new(0,6)

local slider=Instance.new("TextBox",frame)
slider.Size=UDim2.new(0,60,0,30)
slider.Position=UDim2.new(0,160,0,50)
slider.Text="150"
slider.BackgroundColor3=Color3.fromRGB(30,30,30)
slider.TextColor3=Color3.new(1,1,1)
slider.Parent=frame

Instance.new("UICorner",slider).CornerRadius=UDim.new(0,6)

local espToggle=Instance.new("TextButton",frame)
espToggle.Size=UDim2.new(0,120,0,30)
espToggle.Position=UDim2.new(0,20,0,100)
espToggle.Text="透视 NPC: OFF"
espToggle.BackgroundColor3=Color3.fromRGB(40,40,40)
espToggle.TextColor3=Color3.new(1,1,1)
espToggle.Parent=frame

Instance.new("UICorner",espToggle).CornerRadius=UDim.new(0,6)

local aimEnabled=false
local espEnabled=false
local distance=150
local connection

local function cleanup()
	aimEnabled=false
	espEnabled=false
	if connection then
		connection:Disconnect()
	end
	for _,npc in pairs(workspace:GetDescendants()) do
		local hl=npc:FindFirstChild("ESP_Highlight")
		if hl then
			hl:Destroy()
		end
	end
end

close.MouseButton1Click:Connect(function()
	cleanup()
	gui:Destroy()
end)

minimize.MouseButton1Click:Connect(function()
	frame.Visible=false
	miniIcon.Visible=true
end)

miniIcon.MouseButton1Click:Connect(function()
	frame.Visible=true
	miniIcon.Visible=false
end)

slider.FocusLost:Connect(function()
	local v=tonumber(slider.Text)
	if v then
		distance=math.clamp(v,0,300)
	end
end)

aimToggle.MouseButton1Click:Connect(function()
	aimEnabled=not aimEnabled
	aimToggle.Text=aimEnabled and "自瞄 NPC: ON" or "自瞄 NPC: OFF"
end)

espToggle.MouseButton1Click:Connect(function()
	espEnabled=not espEnabled
	espToggle.Text=espEnabled and "透视 NPC: ON" or "透视 NPC: OFF"
	if not espEnabled then
		for _,npc in pairs(workspace:GetDescendants()) do
			local hl=npc:FindFirstChild("ESP_Highlight")
			if hl then
				hl:Destroy()
			end
		end
	end
end)

connection=RunService.RenderStepped:Connect(function()
	if not gui.Parent then
		cleanup()
		return
	end
	
	if aimEnabled then
		local closest,dist=nil,distance
		for _,npc in pairs(workspace:GetDescendants()) do
			if npc:IsA("Model") and npc~=player.Character then
				local root=npc:FindFirstChild("HumanoidRootPart")
				local hum=npc:FindFirstChildOfClass("Humanoid")
				if root and hum and hum.Health>0 then
					local d=(camera.CFrame.Position-root.Position).Magnitude
					if d<dist then
						closest=root
						dist=d
					end
				end
			end
		end
		if closest then
			camera.CFrame=CFrame.new(camera.CFrame.Position,closest.Position)
		end
	end
	
	if espEnabled then
		for _,npc in pairs(workspace:GetDescendants()) do
			if npc:IsA("Model") and npc~=player.Character then
				local hum=npc:FindFirstChildOfClass("Humanoid")
				if hum and hum.Health>0 and not npc:FindFirstChild("ESP_Highlight") then
					local hl=Instance.new("Highlight")
					hl.Name="ESP_Highlight"
					hl.FillColor=Color3.fromRGB(255,0,0)
					hl.OutlineColor=Color3.new(1,1,1)
					hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop
					hl.Parent=npc
				end
			end
		end
	end
end)
