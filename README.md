local library = loadstring(game:HttpGet("https://raw.githubusercontent.com/bloodball/-back-ups-for-libs/main/Splix"))()

local window = library:new({
	textsize = 13.5,
	font = Enum.Font.RobotoMono,
	name = "Xenz Hub - By Xenz Cheats",
	color = Color3.fromRGB(128, 0, 128)
})

local tab = window:page({name = "VISUAL"})
local section1 = tab:section({name = "ESP", side = "left", size = 250})

-- Variáveis de controle dos toggles
local tog = false
local showBox = false
local showName = false
local showDistance = false
local showHP = false
local showSkeleton = false

local ESP_FOLDER = {}
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Toggles
section1:toggle({name = "Show Esp", def = false, callback = function(v) tog = v end})
section1:toggle({name = "Show Box", def = false, callback = function(v) showBox = v end})
section1:toggle({name = "Show Nick", def = false, callback = function(v) showName = v end})
section1:toggle({name = "Show Distance", def = false, callback = function(v) showDistance = v end})
section1:toggle({name = "Show HP", def = false, callback = function(v) showHP = v end})
section1:toggle({name = "Show Skeleton", def = false, callback = function(v) showSkeleton = v end})
-- Adicionar personalizador de cor para o esqueleto
section1:colorpicker({
    name = "Skeleton Color",
    def = Color3.fromRGB(255, 255, 255),  -- Cor vermelha padrão
    callback = function(value)
        -- Atualiza a cor de todas as linhas do esqueleto
        for _, playerESP in pairs(ESP_FOLDER) do
            if playerESP.skeleton then
                for _, bone in ipairs(playerESP.skeleton) do
                    bone.line.Color = value  -- Atualiza a cor das linhas do esqueleto
                end
            end
        end
    end
})

-- Criar ESP
local function createESP(player)
    if player == LocalPlayer or ESP_FOLDER[player] then return end

    local box = Drawing.new("Square")
    box.Thickness = 1
    box.Color = Color3.new(0, 1, 0)
    box.Filled = false
    box.Visible = false

    local nameText = Drawing.new("Text")
    nameText.Color = Color3.new(1, 1, 1)
    nameText.Size = 16
    nameText.Center = true
    nameText.Outline = true
    nameText.Visible = false

    local distanceText = Drawing.new("Text")
    distanceText.Color = Color3.new(1, 1, 0)
    distanceText.Size = 14
    distanceText.Center = true
    distanceText.Outline = true
    distanceText.Visible = false

    local hpText = Drawing.new("Text")
    hpText.Color = Color3.new(1, 0, 0)
    hpText.Size = 14
    hpText.Center = true
    hpText.Outline = true
    hpText.Visible = false

    local skeleton = {}
    local bones = {
        {"Head", "UpperTorso"},
        {"UpperTorso", "LowerTorso"},
        {"UpperTorso", "LeftUpperArm"},
        {"UpperTorso", "RightUpperArm"},
        {"LeftUpperArm", "LeftLowerArm"},
        {"RightUpperArm", "RightLowerArm"},
        {"LeftLowerArm", "LeftHand"},
        {"RightLowerArm", "RightHand"},
        {"LowerTorso", "LeftUpperLeg"},
        {"LowerTorso", "RightUpperLeg"},
        {"LeftUpperLeg", "LeftLowerLeg"},
        {"RightUpperLeg", "RightLowerLeg"},
        {"LeftLowerLeg", "LeftFoot"},
        {"RightLowerLeg", "RightFoot"},
    }

    for _, bone in ipairs(bones) do
        local line = Drawing.new("Line")
        line.Color = Color3.new(1, 1, 1)  -- Alterado para branco
        line.Thickness = 1
        line.Visible = false
        table.insert(skeleton, {from = bone[1], to = bone[2], line = line})
    end

    ESP_FOLDER[player] = {
        box = box,
        name = nameText,
        dist = distanceText,
        hp = hpText,
        skeleton = skeleton
    }

    RunService.RenderStepped:Connect(function()
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local hrp = player.Character:FindFirstChild("HumanoidRootPart")
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

            local distance = (Camera.CFrame.Position - hrp.Position).Magnitude
            local scale = math.clamp(1 / (distance * 0.03), 0.5, 2)
            local boxSize = Vector2.new(60 * scale, 100 * scale)

            box.Position = Vector2.new(pos.X - boxSize.X / 2, pos.Y - boxSize.Y / 2)
            box.Size = boxSize
            box.Visible = tog and showBox and onScreen

            nameText.Position = Vector2.new(pos.X, pos.Y - boxSize.Y / 2 - 16)
            nameText.Text = player.Name
            nameText.Visible = tog and showName and onScreen

            distanceText.Position = Vector2.new(pos.X, pos.Y + boxSize.Y / 2 + 2)
            distanceText.Text = string.format("%.0f m", distance)
            distanceText.Visible = tog and showDistance and onScreen

            hpText.Position = Vector2.new(pos.X, pos.Y + boxSize.Y / 2 + 18)
            hpText.Text = "HP: " .. math.floor(humanoid.Health)
            hpText.Visible = tog and showHP and onScreen

            for _, bone in ipairs(skeleton) do
                local partFrom = player.Character:FindFirstChild(bone.from)
                local partTo = player.Character:FindFirstChild(bone.to)
                if partFrom and partTo then
                    local pos1, vis1 = Camera:WorldToViewportPoint(partFrom.Position)
                    local pos2, vis2 = Camera:WorldToViewportPoint(partTo.Position)
                    bone.line.From = Vector2.new(pos1.X, pos1.Y)
                    bone.line.To = Vector2.new(pos2.X, pos2.Y)
                    bone.line.Visible = tog and showSkeleton and vis1 and vis2
                else
                    bone.line.Visible = false
                end
            end
        else
            box.Visible = false
            nameText.Visible = false
            distanceText.Visible = false
            hpText.Visible = false
            for _, bone in ipairs(skeleton) do
                bone.line.Visible = false
            end
        end
    end)
end

-- Monitorar jogadores
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		wait(1)
		createESP(player)
	end)
end)

for _, player in pairs(Players:GetPlayers()) do
	if player ~= LocalPlayer then
		createESP(player)
	end
end

-- O

local tab = window:page({name = "COMBAT"})
local section1 = tab:section({name = "AIMBOT", side = "left", size = 245})

local AIMBOT_ENABLED = false
local SMOOTHNESS = 0.1 -- Padrão para suavidade
local FOV_ENABLED = false
local FOV_RADIUS = 250 -- Raio padrão para FOV
local TEAM_CHECK = false
local WALL_CHECK = true
local AIM_PART = "Head" -- Parte do corpo que o aimbot vai mirar (Head ou Torso)

local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 2
fovCircle.Color = Color3.fromRGB(255, 255, 255)  -- Cor vermelha
fovCircle.Filled = false

-- Toggle para ativar/desativar o aimbot
section1:toggle({
	name = "Aimbot",
	def = false,
	callback = function(value)
		AIMBOT_ENABLED = value
	end
})

-- Slider para suavidade
section1:slider({
	name = "Smoothness",
	min = 0,
	max = 1,
	def = 0.1,
	callback = function(value)
		SMOOTHNESS = value
	end
})

-- Toggle para ativar/desativar FOV
section1:toggle({
	name = "Show FOV",
	def = false,
	callback = function(value)
		FOV_ENABLED = value
		fovCircle.Visible = FOV_ENABLED  -- Atualiza a visibilidade do círculo FOV
	end
})

-- Slider para configurar o raio do FOV
section1:slider({
	name = "Radius FOV",
	min = 50,
	max = 1000,
	def = 250,
	callback = function(value)
		FOV_RADIUS = value
		fovCircle.Radius = FOV_RADIUS  -- Atualiza o raio do círculo FOV
	end
})

-- Adiciona a funcionalidade de seletor de cor para o FOV
section1:colorpicker({
    name = "FOV Color",
    def = Color3.fromRGB(255, 255, 255),  -- Cor vermelha padrão
    callback = function(value)
        fovCircle.Color = value  -- Atualiza a cor do círculo FOV com a cor selecionada
    end
})

-- Toggle para ativar/desativar TeamCheck
section1:toggle({
	name = "Team Check",
	def = false,
	callback = function(value)
		TEAM_CHECK = value
	end
})

-- Toggle para ativar/desativar WallCheck
section1:toggle({
	name = "Wall Check",
	def = true,
	callback = function(value)
		WALL_CHECK = value
	end
})

-- Dropdown para escolher a parte do corpo para mirar
section1:dropdown({
	name = "Part Aimbot",
	options = {"Head", "Torso"},
	def = "Head",
	callback = function(value)
		AIM_PART = value
	end
})

-- Função para encontrar o jogador mais próximo da mira dentro do FOV
local function getClosestPlayerToMouse()
	local closestPlayer = nil
	local shortestDistance = math.huge
	local mouseLocation = game:GetService("UserInputService"):GetMouseLocation()

	for _, player in pairs(game:GetService("Players"):GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(AIM_PART) then
			-- Verifica se o jogador é da mesma equipe
			if TEAM_CHECK and player.Team == LocalPlayer.Team then
				continue
			end

			local part = player.Character[AIM_PART]
			local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X, pos.Y) - mouseLocation).Magnitude
				if dist < shortestDistance and dist < FOV_RADIUS then -- Dentro do FOV
					shortestDistance = dist
					closestPlayer = player
				end
			end
		end
	end

	return closestPlayer
end

-- Função para verificar se o jogador está visível (WallCheck)
local function isPlayerVisible(player)
	local part = player.Character[AIM_PART]
	local ray = Ray.new(Camera.CFrame.Position, (part.Position - Camera.CFrame.Position).unit * 500)
	local hit, position = workspace:FindPartOnRay(ray, LocalPlayer.Character)

	-- Verifica se atingiu algo que não seja o próprio jogador
	return hit == nil or hit.Parent == player.Character
end

-- Atualiza o círculo de FOV na tela
game:GetService("RunService").RenderStepped:Connect(function()
	-- Atualiza a posição do círculo FOV
	local mouseLocation = game:GetService("UserInputService"):GetMouseLocation()
	fovCircle.Position = mouseLocation
	fovCircle.Visible = FOV_ENABLED  -- Se FOV estiver ativado, o círculo é visível

	if AIMBOT_ENABLED then
		local target = getClosestPlayerToMouse()
		if target and target.Character and target.Character:FindFirstChild(AIM_PART) then
			local part = target.Character[AIM_PART]
			if WALL_CHECK and not isPlayerVisible(target) then
				return
			end

			local headPos = part.Position
			local direction = (headPos - Camera.CFrame.Position).unit
			local newCFrame = Camera.CFrame:Lerp(CFrame.new(Camera.CFrame.Position, headPos), SMOOTHNESS)
			Camera.CFrame = newCFrame
		end
	end
end)

-- Player Categoria

local tab = window:page({name = "PLAYER"})
local section1 = tab:section({name = "WALKSPEED", side = "left", size = 170})

local walkSpeedEnabled = false
local walkSpeed = 16  -- Valor padrão de velocidade (sem alterações)

-- Toggle para habilitar/desabilitar alteração de velocidade
section1:toggle({
    name = "WalkSpeed",
    def = false,
    callback = function(value)
        walkSpeedEnabled = value
        if not walkSpeedEnabled then
            -- Restaura a velocidade padrão se o toggle for desativado
            LocalPlayer.Character.Humanoid.WalkSpeed = 16
        end
    end
})

-- Slider para controlar a velocidade
section1:slider({
    name = "Speed",
    min = 16,
    max = 1000,
    def = 16,
    callback = function(value)
        walkSpeed = value
        if walkSpeedEnabled then
            LocalPlayer.Character.Humanoid.WalkSpeed = walkSpeed
        end
    end
})

-- Atualizar a velocidade do jogador durante o jogo
game:GetService("RunService").RenderStepped:Connect(function()
    if walkSpeedEnabled then
        LocalPlayer.Character.Humanoid.WalkSpeed = walkSpeed
    end
end)

local section1 = tab:section({name = "POWERJUMP", side = "left", size = 170})

local powerJumpEnabled = false
local powerJumpValue = 50 -- Valor padrão do PowerJump

-- Toggle para habilitar/desabilitar o PowerJump
section1:toggle({
    name = "PowerJump",
    def = false,
    callback = function(value)
        powerJumpEnabled = value
        if not powerJumpEnabled then
            -- Restaura o pulo normal se o toggle for desativado
            game:GetService("Players").LocalPlayer.Character.Humanoid.JumpPower = 50
        end
    end
})

-- Slider para controlar o PowerJump
section1:slider({
    name = "PowerJump",
    min = 1,
    max = 500,
    def = 50,
    callback = function(value)
        powerJumpValue = value
        if powerJumpEnabled then
            game:GetService("Players").LocalPlayer.Character.Humanoid.JumpPower = powerJumpValue
        end
    end
})

-- Atualizar o PowerJump durante o jogo
game:GetService("RunService").RenderStepped:Connect(function()
    if powerJumpEnabled then
        game:GetService("Players").LocalPlayer.Character.Humanoid.JumpPower = powerJumpValue
    end
end)

local section1 = tab:section({name = "GRAVITY", side = "right", size = 170})

local gravityEnabled = false
local gravityValue = 196.2  -- Valor padrão de gravidade (gravidade normal no Roblox)

-- Toggle para habilitar/desabilitar alteração de gravidade
section1:toggle({
    name = "Gravity",
    def = false,
    callback = function(value)
        gravityEnabled = value
        if not gravityEnabled then
            -- Restaura a gravidade padrão se o toggle for desativado
            workspace.Gravity = 196.2
        end
    end
})

-- Slider para controlar a gravidade
section1:slider({
    name = "Gravity",
    min = 1,
    max = 500,
    def = 196.2,
    callback = function(value)
        gravityValue = value
        if gravityEnabled then
            workspace.Gravity = gravityValue
        end
    end
})

-- Atualizar a gravidade do jogo durante o jogo
game:GetService("RunService").RenderStepped:Connect(function()
    if gravityEnabled then
        workspace.Gravity = gravityValue
    end
end)

local tab = window:page({name = "MISC"})
local section1 = tab:section({name = "MISC", side = "left", size = 250})

local showFPS = false
local fpsText = Drawing.new("Text")
fpsText.Color = Color3.new(1, 1, 1)
fpsText.Size = 16
fpsText.Position = Vector2.new(10, 10)  -- Position in the top left corner
fpsText.Outline = true
fpsText.Visible = false

-- Toggle to enable/disable FPS display
section1:toggle({
    name = "Show FPS",
    def = false,
    callback = function(value)
        showFPS = value
        fpsText.Visible = showFPS
    end
})

-- Update the FPS every frame
game:GetService("RunService").RenderStepped:Connect(function()
    if showFPS then
        local fps = math.floor(1 / game:GetService("RunService").Heartbeat:Wait())  -- Calculate FPS
        fpsText.Text = "FPS: " .. fps
    end
end)
