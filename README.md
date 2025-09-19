
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local character = player.Character or player.CharacterAdded:Wait()

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "EnemyRainGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 200, 0, 50)
button.Position = UDim2.new(0.5, -100, 0.9, -25)
button.Text = "Chuva de Inimigos ☁️"
button.Parent = screenGui
button.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
button.TextScaled = true
button.Font = Enum.Font.SourceSansBold
button.TextColor3 = Color3.new(1,1,1)

local function createEnemyTemplate()
	local model = Instance.new("Model")
	model.Name = "EnemyNPC"

	local part = Instance.new("Part")
	part.Size = Vector3.new(2, 2, 1)
	part.Anchored = false
	part.BrickColor = BrickColor.new("Bright red")
	part.Position = Vector3.new(0, 100, 0)
	part.Name = "HumanoidRootPart"
	part.CanCollide = true
	part.Parent = model

	local humanoid = Instance.new("Humanoid")
	humanoid.WalkSpeed = 12
	humanoid.Parent = model

	model.PrimaryPart = part

	return model
end

local enemyTemplate = createEnemyTemplate()

local raining = false
local rainThread

local function makeEnemyAttack(enemy)
	local enemyHumanoid = enemy:FindFirstChildOfClass("Humanoid")
	local enemyRoot = enemy:FindFirstChild("HumanoidRootPart")

	if not (enemyHumanoid and enemyRoot) then return end

	local function moveToPlayer()
		while enemyHumanoid.Health > 0 and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
			local targetPos = player.Character.HumanoidRootPart.Position
			enemyHumanoid:MoveTo(targetPos)
			wait(1)
		end
	end

	coroutine.wrap(moveToPlayer)()

	local touchedConn
	touchedConn = enemyRoot.Touched:Connect(function(hit)
		if hit:IsDescendantOf(player.Character) then
			local playerHumanoid = player.Character:FindFirstChildOfClass("Humanoid")
			if playerHumanoid and playerHumanoid.Health > 0 then
				playerHumanoid:TakeDamage(10)
			end
		end
	end)

	enemyHumanoid.Died:Connect(function()
		if touchedConn then
			touchedConn:Disconnect()
		end
	end)
end

local function spawnEnemy()
	local enemy = enemyTemplate:Clone()
	local x = math.random(-100, 100)
	local z = math.random(-100, 100)
	local y = 100

	if enemy.PrimaryPart then
		enemy:SetPrimaryPartCFrame(CFrame.new(x, y, z))
		enemy.Parent = workspace
		makeEnemyAttack(enemy)
	end
end

local function startRain()
	raining = true
	rainThread = coroutine.create(function()
		while raining do
			spawnEnemy()
			wait(1)
		end
	end)
	coroutine.resume(rainThread)
end

local function stopRain()
	raining = false
end

button.MouseButton1Click:Connect(function()
	if raining then
		stopRain()
		button.Text = "Chuva de Inimigos ☁️"
		button.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
	else
		startRain()
		button.Text = "Parar Chuva 🌤️"
		button.BackgroundColor3 = Color3.fromRGB(80, 255, 80)
	end
end)
