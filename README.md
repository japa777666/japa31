local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/jensonhirst/Orion/main/source')))()

local Window = OrionLib:MakeWindow({Name = "💎Japa Menu V3.3", HidePremium = false, SaveConfig = true, ConfigFolder = "OrionTest"})

-- Variáveis de controle
local flyEnabled = false
local isFlying = false
local speed = 20
local bodyVelocity, bodyGyro, connection
local aimbotEnabled = false
local aimSensitivity = 0.3
local fovSize = 100
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Thickness = 2
fovCircle.NumSides = 100
fovCircle.Radius = fovSize
fovCircle.Filled = false
fovCircle.Visible = false
local aimDistance = 100 -- Valor padrão
local AimbotEnabled = false
local BoxESPEnabled = false
local NameTagsEnabled = false
local ESPObjects = {}
local NameTags = {}
local plr = game.Players.LocalPlayer
local nameTagSize = 17.5
local nameTagColor = Color3.fromRGB(255, 255, 255)
local boxColor = Color3.fromRGB(255, 255, 255)
local chamFillColor = Color3.fromRGB(175, 25, 255)
local chamOutlineColor = Color3.fromRGB(255, 255, 255)
local fovColor = Color3.fromRGB(255, 255, 255)
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local aimbotKeybind = Enum.KeyCode.Q
local aimbotHolding = false

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:FindFirstChildOfClass("Humanoid")

local freecamEnabled = false
local cameraOffset = Vector3.new(0, 5, 10)
local mouseDelta = Vector2.new(0, 0)
local mouseMovementConnection
local teleportConnection
local renderSteppedConnection
local moveSpeed = 0.5
local rotationSpeedQ = 0.001 -- Esta variável agora controla todas as rotações

-- Variáveis essenciais
local Player = game:GetService("Players").LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
local Camera = game:GetService("Workspace").CurrentCamera
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local function onMouseMove(input)
    if freecamEnabled and input.UserInputType == Enum.UserInputType.MouseMovement then
        mouseDelta = input.Delta
    end
end

local function enableFreecam()
    if not freecamEnabled then
        freecamEnabled = true
        Camera.CameraType = Enum.CameraType.Scriptable
        Camera.CFrame = HumanoidRootPart.CFrame * CFrame.new(cameraOffset)
        
        -- Desabilitar movimentação do personagem
        if Humanoid then
            Humanoid.WalkSpeed = 0
            Humanoid.JumpPower = 0
            Humanoid.PlatformStand = true
        end
        
        -- Conexão para teleporte
        teleportConnection = UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
            if freecamEnabled and not gameProcessedEvent then
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    -- Cálculo do teleporte
                    local rayOrigin = Camera.CFrame.Position
                    local rayDirection = Camera.CFrame.LookVector * 1000
                    
                    local raycastParams = RaycastParams.new()
                    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
                    raycastParams.FilterDescendantsInstances = {Character}
                    
                    local raycastResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
                    
                    if raycastResult then
                        HumanoidRootPart.CFrame = CFrame.new(raycastResult.Position + Vector3.new(0, 3, 0))
                    else
                        HumanoidRootPart.CFrame = CFrame.new(rayOrigin + (Camera.CFrame.LookVector * 50))
                    end
                end
            end
        end)

        mouseMovementConnection = UserInputService.InputChanged:Connect(onMouseMove)
        
        renderSteppedConnection = RunService.RenderStepped:Connect(function()
            if freecamEnabled then
                local moveDirection = Vector3.new(0, 0, 0)
                
                -- Controles de movimento
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDirection = moveDirection + Vector3.new(0, 0, -1) end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDirection = moveDirection + Vector3.new(0, 0, 1) end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDirection = moveDirection + Vector3.new(-1, 0, 0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDirection = moveDirection + Vector3.new(1, 0, 0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDirection = moveDirection + Vector3.new(0, -1, 0) end
                
                -- Aplicar movimento
                Camera.CFrame = Camera.CFrame * CFrame.new(moveDirection * moveSpeed)
                
                -- Rotação horizontal (Q/E)
                if UserInputService:IsKeyDown(Enum.KeyCode.Q) then
                    Camera.CFrame = Camera.CFrame * CFrame.Angles(0, rotationSpeedQ, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.E) then
                    Camera.CFrame = Camera.CFrame * CFrame.Angles(0, -rotationSpeedQ, 0)
                end
                
                -- Rotação vertical (Z/X) - Adicionado aqui
                if UserInputService:IsKeyDown(Enum.KeyCode.Z) then
                    Camera.CFrame = Camera.CFrame * CFrame.Angles(rotationSpeedQ, 0, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.X) then
                    Camera.CFrame = Camera.CFrame * CFrame.Angles(-rotationSpeedQ, 0, 0)
                end
                
                -- Rotação com mouse
                if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                    local sensitivity = 0.1
                    Camera.CFrame = Camera.CFrame * 
                        CFrame.Angles(0, -mouseDelta.X * sensitivity, 0) * 
                        CFrame.Angles(-mouseDelta.Y * sensitivity, 0, 0)
                    mouseDelta = Vector2.new(0, 0)
                end
            end
        end)
    end
end

local function disableFreecam()
    freecamEnabled = false
    Camera.CameraType = Enum.CameraType.Custom
    Camera.CFrame = HumanoidRootPart.CFrame
    
    -- Desconectar conexões
    if mouseMovementConnection then mouseMovementConnection:Disconnect() end
    if teleportConnection then teleportConnection:Disconnect() end
    if renderSteppedConnection then renderSteppedConnection:Disconnect() end
    
    -- Restaurar personagem
    if Humanoid then
        Humanoid.WalkSpeed = 16
        Humanoid.JumpPower = 50
        Humanoid.PlatformStand = false
    end
end

-- Sistema de ativação (adicione seu próprio método preferido)
UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
    if not gameProcessedEvent then
        if input.KeyCode == Enum.KeyCode.F then
            if freecamEnabled then disableFreecam() else enableFreecam() end
        end
    end
end)
-- Funções para Box ESP
local function CreateBox()
    local box = Drawing.new("Square")
    box.Thickness = 2.5
    box.Color = boxColor
    box.Filled = false
    box.Visible = false
    return box
end

local function UpdateBox(box, rootPart)
    local viewportPoint, onScreen = workspace.CurrentCamera:WorldToViewportPoint(rootPart.Position)
    if onScreen then
        local size = Vector2.new(2000 / viewportPoint.Z, 4000 / viewportPoint.Z)
        box.Size = size
        box.Position = Vector2.new(viewportPoint.X - size.X / 2, viewportPoint.Y - size.Y / 2)
        box.Visible = true
    else
        box.Visible = false
    end
end

local function HandleExistingPlayersESP()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local box = CreateBox()
            ESPObjects[player.Name] = box
            coroutine.wrap(function()
                while BoxESPEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
                    UpdateBox(box, player.Character.HumanoidRootPart)
                    wait(0.01) -- Atualização mais rápida
                end
                if ESPObjects[player.Name] then
                    ESPObjects[player.Name]:Remove()
                    ESPObjects[player.Name] = nil
                end
            end)()
        end
    end
end

local function ToggleBoxESP(state)
    BoxESPEnabled = state
    if BoxESPEnabled then
        HandleExistingPlayersESP() -- Lida com jogadores já existentes
        game.Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function(character)
                if character:FindFirstChild("HumanoidRootPart") then
                    local box = CreateBox()
                    ESPObjects[player.Name] = box
                    coroutine.wrap(function()
                        while BoxESPEnabled and character:FindFirstChild("HumanoidRootPart") do
                            UpdateBox(box, character.HumanoidRootPart)
                            wait(0.03)
                        end
                        if ESPObjects[player.Name] then
                            ESPObjects[player.Name]:Remove()
                            ESPObjects[player.Name] = nil
                        end
                    end)()
                end
            end)
        end)
    else
        for _, box in pairs(ESPObjects) do
            box:Remove()
        end
        ESPObjects = {}
    end
end

-- Funções para Nametags
local function CreateNameTag()
    local nametag = Drawing.new("Text")
    nametag.Size = nameTagSize
    nametag.Color = nameTagColor
    nametag.Visible = false
    return nametag
end


local function UpdateNameTag(nametag, rootPart)
    local viewportPoint, onScreen = workspace.CurrentCamera:WorldToViewportPoint(rootPart.Position)
    if onScreen then
        nametag.Position = Vector2.new(viewportPoint.X, viewportPoint.Y - 20)
        nametag.Visible = true
    else
        nametag.Visible = false
    end
end

local function HandleExistingPlayersNameTags()
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local nametag = CreateNameTag()
            NameTags[player.Name] = nametag
            coroutine.wrap(function()
                while NameTagsEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") do
                    nametag.Text = player.Name
                    UpdateNameTag(nametag, player.Character.HumanoidRootPart)
                    wait(0.01)
                end
                if NameTags[player.Name] then
                    NameTags[player.Name]:Remove()
                    NameTags[player.Name] = nil
                end
            end)()
        end
    end
end

local function ToggleNameTags(state)
    NameTagsEnabled = state
    if NameTagsEnabled then
        HandleExistingPlayersNameTags() -- Lida com jogadores já existentes
        game.Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function(character)
                if character:FindFirstChild("HumanoidRootPart") then
                    local nametag = CreateNameTag()
                    NameTags[player.Name] = nametag
                    coroutine.wrap(function()
                        while NameTagsEnabled and character:FindFirstChild("HumanoidRootPart") do
                            nametag.Text = player.Name
                            UpdateNameTag(nametag, character.HumanoidRootPart)
                            wait(0.03)
                        end
                        if NameTags[player.Name] then
                            NameTags[player.Name]:Remove()
                            NameTags[player.Name] = nil
                        end
                    end)()
                end
            end)
        end)
    else
        for _, nametag in pairs(NameTags) do
            nametag:Remove()
        end
        NameTags = {}
    end
end

function updateFovCircle()
    fovCircle.Radius = fovSize
    fovCircle.Position = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y / 2)
    fovCircle.Color = fovColor -- Garante que a cor seja atualizada
end



-- Modificar a função enableAimbot para:
function enableAimbot(state)
    aimbotEnabled = state
    fovCircle.Visible = state
    
    if aimbotEnabled then
        game:GetService("RunService").RenderStepped:Connect(function()
            if aimbotEnabled and aimbotHolding then  -- Só ativa quando segurando a bind
                local camera = workspace.CurrentCamera
                local closestTarget = nil
                local shortestDistance = math.huge

                for _, player in pairs(game.Players:GetPlayers()) do
                    if player ~= plr and player.Character and player.Character:FindFirstChild("Head") then
                        local head = player.Character.Head
                        local screenPoint, onScreen = camera:WorldToScreenPoint(head.Position)
                        local mousePosition = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
                        local distanceFromMouse = (Vector2.new(screenPoint.X, screenPoint.Y) - mousePosition).Magnitude

                        local playerDistance = (head.Position - camera.CFrame.Position).Magnitude
                        if onScreen and distanceFromMouse < fovSize and distanceFromMouse < shortestDistance and playerDistance <= aimDistance then
                            shortestDistance = distanceFromMouse
                            closestTarget = head
                        end
                    end
                end

                if closestTarget then
                    local targetPosition = closestTarget.Position
                    local aimDirection = (targetPosition - camera.CFrame.Position).unit
                    camera.CFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + aimDirection:Lerp(camera.CFrame.LookVector, aimSensitivity))
                end
            end
        end)
    end
end


local function enableFly()
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")
    local root = character:WaitForChild("HumanoidRootPart")

    humanoid.PlatformStand = true

    -- Cria controles físicos
    bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = Vector3.new()
    bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bodyVelocity.P = 1000
    bodyVelocity.Parent = root

    bodyGyro = Instance.new("BodyGyro")
    bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bodyGyro.P = 1000
    bodyGyro.D = 100
    bodyGyro.CFrame = root.CFrame
    bodyGyro.Parent = root

    -- Conexão principal para voo e noclip
    connection = game:GetService("RunService").Heartbeat:Connect(function()
        if not isFlying then return end
        
        -- Captura direção do movimento
        local cam = workspace.CurrentCamera.CFrame
        local moveDir = Vector3.new()
        
        -- Verifica teclas pressionadas
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.W) then
            moveDir = moveDir + Vector3.new(0, 0, -1)
        end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.S) then
            moveDir = moveDir + Vector3.new(0, 0, 1)
        end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.A) then
            moveDir = moveDir + Vector3.new(-1, 0, 0)
        end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.D) then
            moveDir = moveDir + Vector3.new(1, 0, 0)
        end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.Space) then
            moveDir = moveDir + Vector3.new(0, 1, 0)
        end
        if game:GetService("UserInputService"):IsKeyDown(Enum.KeyCode.LeftShift) then
            moveDir = moveDir + Vector3.new(0, -1, 0)
        end

        -- Aplica velocidade
        local velocity = cam:VectorToWorldSpace(moveDir) * speed
        bodyVelocity.Velocity = velocity

        -- Noclip
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    end)
end

local function disableFly()
    if connection then
        connection:Disconnect()
        connection = nil
    end
    
    local player = game.Players.LocalPlayer
    local character = player.Character
    if character then
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.PlatformStand = false
        end
        
        local root = character:FindFirstChild("HumanoidRootPart")
        if root then
            if bodyVelocity then
                bodyVelocity:Destroy()
                bodyVelocity = nil
            end
            if bodyGyro then
                bodyGyro:Destroy()
                bodyGyro = nil
            end
        end
        
        -- Reativa colisões
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end

local Tab = Window:MakeTab({
    Name = "🙍Self",
    Icon = "rbxassetid://",
    PremiumOnly = false
})


OrionLib:MakeNotification({
    Name = "Japa Menu V3",
    Content = "Menu injetado Com Sucesso!",
    Image = "rbxassetid://4483345998",
    Time = 5
})

local WallTab = Window:MakeTab({
    Name = "👁️Visuals",
    Icon = "rbxassetid://",
    PremiumOnly = false
})

local Section = WallTab:AddSection({
    Name = "Esp"
})

local AimbotTab = Window:MakeTab({
    Name = "🔫Aimbot",
    Icon = "",
    PremiumOnly = false
})

local Section = AimbotTab:AddSection({
    Name = "Aimbot"
})

local Section = Tab:AddSection({
    Name = "Self"
})

-- Adicionar no início com as outras variáveis
local spectatingPlayer = nil
local spectateEnabled = false
local spectateDistance = 10
local spectateConnection = nil
local spectateRotation = 0
local baseSpectateOffset = Vector3.new(0, 0, 0) -- Offset base dentro da cabeça


-- Nova aba Players
local PlayersTab = Window:MakeTab({
    Name = "👪Players",
    Icon = "rbxassetid://",
    PremiumOnly = false
})

-- Adicionar no início com as outras variáveis
local spectatingPlayer = nil
local spectateEnabled = false
local spectateDistance = 10
local spectateConnection = nil
local selectedPlayer = nil
local spectateMouseConnection = nil
local originalCameraType = Enum.CameraType.Custom

-- Container para os elementos dinâmicos
local playerListContainer = {}

-- Função para atualizar a lista de jogadores
local function UpdatePlayerList()
    -- Limpar apenas os elementos da lista
    for _, v in ipairs(playerListContainer) do
        v:Destroy()
    end
    playerListContainer = {}

    -- Obter jogadores (exceto o local) e ordenar por nome
    local players = {}
    for _, player in ipairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            table.insert(players, player)
        end
    end
    table.sort(players, function(a, b)
        return a.Name:lower() < b.Name:lower()
    end)

    -- Criar botões ordenados
    for _, player in ipairs(players) do
        local btn = PlayersTab:AddButton({
            Name = player.Name,
            Callback = function()
                selectedPlayer = player
                SpectatePlayer() -- <== ADICIONE ISSO AQUI
                OrionLib:MakeNotification({
                    Name = "Japa Menu",
                    Content = "Selecionado: "..player.Name,
                    Image = "rbxassetid://4483345998",
                    Time = 2
                })
            end
        })
        table.insert(playerListContainer, btn)
    end
end

-- Função para espectar o jogador (substituir a anterior)
local function SpectatePlayer()
    if spectateConnection then
        spectateConnection:Disconnect()
    end
    
    spectateConnection = game:GetService("RunService").RenderStepped:Connect(function()
        if spectateEnabled and selectedPlayer and selectedPlayer.Character then
            local targetChar = selectedPlayer.Character
            local head = targetChar:FindFirstChild("Head")
            local root = targetChar:FindFirstChild("HumanoidRootPart")
            
            if head and root then
                -- Calcula a direção da câmera com rotação
                local cameraDirection = CFrame.Angles(0, math.rad(spectateRotation), 0)
                
                -- Offset base + distância
                local offset = cameraDirection * CFrame.new(0, 0, -spectateDistance)
                
                -- Posição final da câmera
                local cameraPos = head.CFrame:ToWorldSpace(offset).Position
                
                -- Mantém o foco na cabeça
                workspace.CurrentCamera.CFrame = CFrame.new(cameraPos, head.Position)
            end
        end
    end)
end

-- Adicione esta nova versão:
local qDown = false
local eDown = false
local rotationSpeed = 2 -- Ajuste a velocidade conforme necessário

UserInputService.InputBegan:Connect(function(input)
    if spectateEnabled then
        if input.KeyCode == Enum.KeyCode.Q then
            qDown = true
        elseif input.KeyCode == Enum.KeyCode.E then
            eDown = true
        end
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Q then
        qDown = false
    elseif input.KeyCode == Enum.KeyCode.E then
        eDown = false
    end
end)

RunService.RenderStepped:Connect(function(deltaTime)
    if spectateEnabled then
        if qDown then
            spectateRotation = spectateRotation - (rotationSpeed * deltaTime * 60)
        end
        if eDown then
            spectateRotation = spectateRotation + (rotationSpeed * deltaTime * 60)
        end
    end
end)

local Section = PlayersTab:AddSection({
    Name = "opções"
})

PlayersTab:AddButton({
    Name = "Teleportar para Player",
    Callback = function()
        if selectedPlayer and selectedPlayer.Character then
            local targetPos = selectedPlayer.Character.HumanoidRootPart.Position
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(targetPos + Vector3.new(0, 3, 0))
        end
    end
})

-- Variável que controla o estado do teleporte em loop
local tpCheckboxChecked = false

-- Adicionando o toggle para o teleporte em loop
PlayersTab:AddToggle({
    Name = "Teleporte em Loop",
    Default = false, -- Estado inicial (desmarcado)
    Callback = function(value)
        tpCheckboxChecked = value  -- Atualiza o estado da checkbox
    end
})

-- Função que é executada em loop enquanto a checkbox estiver marcada
local function teleportLoop()
    while true do
        if tpCheckboxChecked then
            if selectedPlayer and selectedPlayer.Character then
                local targetPos = selectedPlayer.Character.HumanoidRootPart.Position
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(targetPos + Vector3.new(0, 3, 0))
            end
        end
        wait(0.01) -- Intervalo entre as verificações (ajustável conforme necessário)
    end
end

-- Iniciar o loop de teleporte em segundo plano
spawn(teleportLoop)

-- Atualizar o toggle de espectar
PlayersTab:AddToggle({
    Name = "Espectar Player",
    Default = false,
    Callback = function(value)
        spectateEnabled = value
        if value and selectedPlayer then
            workspace.CurrentCamera.CameraType = Enum.CameraType.Scriptable
            SpectatePlayer()
        else
            if spectateConnection then
                spectateConnection:Disconnect()
            end
            workspace.CurrentCamera.CameraType = Enum.CameraType.Custom
            spectateRotation = 0 -- Resetar rotação
        end
    end
})

-- Atualizar o slider na UI
PlayersTab:AddSlider({
    Name = "Distância da Câmera",
    Min = 0, -- 0 = Dentro da cabeça
    Max = 50,
    Default = 0,
    Color = Color3.fromRGB(119, 18, 169),
    Increment = 1,
    ValueName = "Metros",
    Callback = function(value)
        spectateDistance = value
    end
})


local mainSection = PlayersTab:AddSection({Name = "Lista de Jogadores"})

-- Inicialização
-- Atualização automática quando jogador entra ou sai
game.Players.PlayerAdded:Connect(UpdatePlayerList)
game.Players.PlayerRemoving:Connect(UpdatePlayerList)
UpdatePlayerList()

-- Observador para novos jogadores
game.Players.PlayerAdded:Connect(UpdatePlayerList)
game.Players.PlayerRemoving:Connect(UpdatePlayerList)

local ExploitsTab = Window:MakeTab({
    Name = "💻Exploits",
    Icon = "rbxassetid://",
    PremiumOnly = false
})

-- Seções da aba Exploits
local sectionVoice = ExploitsTab:AddSection({ Name = "Voice" })

-- Botão: Voltar Ao Voice
ExploitsTab:AddButton({
    Name = "Voltar Ao Voice",
    Callback = function()
        local vci = cloneref and cloneref(game:GetService("VoiceChatInternal"))
        local vcs = cloneref and cloneref(game:GetService("VoiceChatService"))

        if vci and vcs then
            local success, err = pcall(function()
                vci:Leave()
                task.wait(0.2)
                vcs:rejoinVoice()
                vcs:joinVoice()
            end)

            if success then
                print("✅ Voice reconectado com sucesso.")
                game.StarterGui:SetCore("SendNotification", {
                    Title = "Voice Chat",
                    Text = "Reconectado com sucesso!",
                    Duration = 3
                })
            else
                warn("❌ Erro ao reconectar:", err)
                game.StarterGui:SetCore("SendNotification", {
                    Title = "Voice Chat",
                    Text = "Erro ao reconectar.",
                    Duration = 3
                })
            end
        else
            print("❌ VoiceChatService não disponível neste jogo.")
            game.StarterGui:SetCore("SendNotification", {
                Title = "Voice Chat",
                Text = "VoiceChatService indisponível.",
                Duration = 3
            })
        end
    end
})

local sectionVisual = ExploitsTab:AddSection({ Name = "Shaders " })

local shadersAtivos = false
local lighting = game:GetService("Lighting")

ExploitsTab:AddToggle({
    Name = "Ativar Shaders",
    Default = false,
    Callback = function(estado)
        shadersAtivos = estado

        if estado then
            -- Color Correction (suave e sem laranja)
            local cc = Instance.new("ColorCorrectionEffect", lighting)
            cc.Name = "JapaColor"
            cc.Brightness = 0.05
            cc.Contrast = 0.2
            cc.Saturation = 0.3
            cc.TintColor = Color3.fromRGB(240, 240, 255) -- Azul claro levemente frio

            -- Bloom (brilho suave)
            local bloom = Instance.new("BloomEffect", lighting)
            bloom.Name = "JapaBloom"
            bloom.Intensity = 0.25
            bloom.Threshold = 0.8
            bloom.Size = 64

            -- Depth of Field (foco de câmera)
            local dof = Instance.new("DepthOfFieldEffect", lighting)
            dof.Name = "JapaDOF"
            dof.FarIntensity = 0.2
            dof.FocusDistance = 35
            dof.InFocusRadius = 50
            dof.NearIntensity = 0.1

            -- Luz ambiente refinada
            lighting.Ambient = Color3.fromRGB(100, 100, 120)
            lighting.OutdoorAmbient = Color3.fromRGB(130, 130, 145)
            lighting.Brightness = 3

            print("✨ Shaders estilosos ativados.")
        else
            -- Remover efeitos
            for _, name in ipairs({"JapaColor", "JapaBloom", "JapaDOF"}) do
                local e = lighting:FindFirstChild(name)
                if e then e:Destroy() end
            end

            -- Restaurar iluminação padrão
            lighting.Ambient = Color3.fromRGB(127, 127, 127)
            lighting.OutdoorAmbient = Color3.fromRGB(127, 127, 127)
            lighting.Brightness = 2

            print("❌ Shaders desativados.")
        end
    end
})

local sectionPuxar = ExploitsTab:AddSection({ Name = "Puxar Players" })

-- Funções auxiliares
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")

local function safeExecute(func)
    local success, result = pcall(func)
    if not success then
        warn("Erro: " .. result)
    end
end

local function teleportAllPlayers()
    local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local otherHRP = player.Character.HumanoidRootPart
            otherHRP.Anchored = true
            otherHRP.CanCollide = false
            otherHRP.CFrame = hrp.CFrame * CFrame.new(math.random(-5,5), 0, math.random(-5,5))
        end
    end
end

local function unanchorAllPlayers()
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = player.Character.HumanoidRootPart
            hrp.Anchored = false
            hrp.CanCollide = true
        end
    end
end

-- Botão: Puxar jogadores
ExploitsTab:AddButton({
    Name = "Puxar Todos os Jogadores",
    Callback = function()
        safeExecute(teleportAllPlayers)
        print("Jogadores puxados para perto.")
    end
})

-- Botão: Destravar jogadores
ExploitsTab:AddButton({
    Name = "Destravar Jogadores",
    Callback = function()
        safeExecute(unanchorAllPlayers)
        print("Jogadores destravados.")
    end
})

local ConfigTab = Window:MakeTab({
    Name = "⚙️Misc",
    Icon = "rbxassetid://",
    PremiumOnly = false
})

local Section = ConfigTab:AddSection({
    Name = "Freecam"
})

ConfigTab:AddToggle({
    Name = "Ativar Freecam",
    Default = false,
    Callback = function(value)
        if value then
            enableFreecam()
        else
            disableFreecam()
        end
    end
})

ConfigTab:AddSlider({
    Name = "Velocidade Freecam",
    Min = 0.5,
    Max = 5,
    Default = 1,
    Increment = 0.5,
    Callback = function(value)
        moveSpeed = value
    end
})

ConfigTab:AddSlider({
    Name = "Velocidade Rotação (Q / E)",
    Min = 0.01,
    Max = 0.1,
    Default = 0.01,
    Increment = 0.001,
    Callback = function(value)
        rotationSpeedQ = value
    end
})

local Section = ConfigTab:AddSection({
    Name = "Configurações"
})

local mouseLocked = false
local UIS = game:GetService("UserInputService")

-- Começa destravado
UIS.MouseBehavior = Enum.MouseBehavior.Default
UIS.MouseIconEnabled = true

-- Tira da primeira pessoa
local function resetCamera()
    local camera = workspace.CurrentCamera
    local player = game.Players.LocalPlayer
    if player and player.Character and player.Character:FindFirstChild("Humanoid") then
        camera.CameraSubject = player.Character:FindFirstChild("Humanoid")
        camera.CameraType = Enum.CameraType.Custom
        camera.CFrame = camera.CFrame * CFrame.new(0, 0, 3)
    end
end

-- Travar/destravar o mouse
local function toggleMouseLock()
    mouseLocked = not mouseLocked
    if mouseLocked then
        UIS.MouseBehavior = Enum.MouseBehavior.LockCenter
        UIS.MouseIconEnabled = false
    else
        UIS.MouseBehavior = Enum.MouseBehavior.Default
        UIS.MouseIconEnabled = true
        resetCamera()
    end
end

-- Removido: InputBegan com lockKey

-- Bind configurável
ConfigTab:AddBind({
    Name = "Travar/Destravar O Mouse",
    Default = Enum.KeyCode.V,
    Hold = false,
    Callback = function()
        toggleMouseLock()
    end
})


ConfigTab:AddButton({
    Name = "Reiniciar Script",
    Callback = function()
        loadstring(game:HttpGet(('https://raw.githubusercontent.com/japa777666/japa31/refs/heads/main/README.md')))() 
    end
})

ConfigTab:AddButton({
    Name = "Voltar Ao Menu Principal",
    Callback = function()
        loadstring(game:HttpGet(('https://raw.githubusercontent.com/japa777666/japaini3333/refs/heads/main/README.md')))() 
    end
})

Tab:AddToggle({
    Name = "Voar",
    Default = false,
    Callback = function(Value)
        flyEnabled = Value
        if not flyEnabled then
            isFlying = false
            disableFly()
        end
    end    
})

Tab:AddBind({
    Name = "Bind Voar",
    Default = Enum.KeyCode.E,
    Hold = false,
    Callback = function()
        if flyEnabled then
            isFlying = not isFlying
            if isFlying then
                enableFly()
            else
                disableFly()
            end
        end
    end    
})

Tab:AddSlider({
    Name = "Velocidade",
    Min = 20,
    Max = 500,
    Default = 20,
    Color = Color3.fromRGB(119, 18, 169),
    Increment = 1,
    ValueName = "Velocidade",
    Callback = function(Value)
        speed = Value
    end    
})

local sectionVoice = Tab:AddSection({ Name = "Teleport Forward" })

-- Função de teleporte para frente
local function teleportForward()
    local player = game.Players.LocalPlayer
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return end

    local hrp = player.Character.HumanoidRootPart
    local lookVector = hrp.CFrame.LookVector
    local distance = 5-- Distância para frente (pode ajustar)

    -- Teleportar na direção que está olhando
    hrp.CFrame = hrp.CFrame + (lookVector * distance)
end

Tab:AddButton({
    Name = "Teleport Forward",
    Callback = teleportForward
})

local sectionVoice = Tab:AddSection({ Name = "Pulos Infinitos" })

-- Infinite Jump lógica
local infiniteJumpEnabled = false
local UserInputService = game:GetService("UserInputService")

UserInputService.JumpRequest:Connect(function()
    if infiniteJumpEnabled then
        local player = game.Players.LocalPlayer
        if player.Character and player.Character:FindFirstChild("Humanoid") then
            player.Character.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end
    end
end)

-- Checkbox para ativar/desativar pulo infinito
Tab:AddToggle({
    Name = "Infinite Jump",
    Default = false,
    Callback = function(state)
        infiniteJumpEnabled = state
    end
})

function esp(enabled)
    if not enabled then
        if game.CoreGui:FindFirstChild("Highlight_Storage") then
            game.CoreGui.Highlight_Storage:Destroy()
        end
        return
    end

    -- Usar variáveis globais para as cores
    local FillColor = chamFillColor
    local OutlineColor = chamOutlineColor

    local DepthMode = "AlwaysOnTop"
    local FillTransparency = 0.5
    local OutlineTransparency = 0

    local CoreGui = game:FindService("CoreGui")
    local Players = game:FindService("Players")
    local lp = Players.LocalPlayer
    local connections = {}

    local Storage = Instance.new("Folder")
    Storage.Parent = CoreGui
    Storage.Name = "Highlight_Storage"

    local function Highlight(plr)
        local Highlight = Instance.new("Highlight")
        Highlight.Name = plr.Name
        Highlight.FillColor = FillColor
        Highlight.DepthMode = DepthMode
        Highlight.FillTransparency = FillTransparency
        Highlight.OutlineColor = OutlineColor
        Highlight.OutlineTransparency = OutlineTransparency
        Highlight.Parent = Storage
        
        local plrchar = plr.Character
        if plrchar then
            Highlight.Adornee = plrchar
        end

        connections[plr] = plr.CharacterAdded:Connect(function(char)
            Highlight.Adornee = char
        end)
    end

    Players.PlayerAdded:Connect(Highlight)
    for i, v in next, Players:GetPlayers() do
        if v ~= lp then
            Highlight(v)
        end
    end

    Players.PlayerRemoving:Connect(function(plr)
        local plrname = plr.Name
        if Storage:FindFirstChild(plrname) then
            Storage[plrname]:Destroy()
        end
        if connections[plr] then
            connections[plr]:Disconnect()
        end
    end)
end


-- Corrigir os Toggles
WallTab:AddToggle({
    Name = "Nametags",
    Default = false,
    Callback = function(Value)
        ToggleNameTags(Value) -- Corrigido para usar Value
    end
})

WallTab:AddToggle({
    Name = "Box",
    Default = false,
    Callback = function(Value)
        ToggleBoxESP(Value) -- Corrigido para usar Value
    end
})

WallTab:AddSlider({
    Name = "Tamanho das Nametags",
    Min = 10,
    Max = 35,
    Default = 15,
    Color = Color3.fromRGB(119, 18, 169),
    Increment = 1,
    ValueName = "Tamanho",
    Callback = function(Value)
        nameTagSize = Value
        -- Atualizar nametags existentes
        for playerName, nametag in pairs(NameTags) do
            nametag.Size = Value
        end
    end
})

WallTab:AddColorpicker({
    Name = "Cor Do Esp",
    Default = Color3.new(1, 1, 1),
    Callback = function(Value)
        -- Atualizar cores
        nameTagColor = Value
        boxColor = Value
        
        -- Atualizar nametags
        for _, nametag in pairs(NameTags) do
            nametag.Color = Value
        end
        
        -- Atualizar boxes
        for _, box in pairs(ESPObjects) do
            box.Color = Value
        end
        
        -- Atualizar Chams
        chamFillColor = Value
        if game.CoreGui:FindFirstChild("Highlight_Storage") then
            for _, highlight in pairs(game.CoreGui.Highlight_Storage:GetChildren()) do
                if highlight:IsA("Highlight") then
                    highlight.FillColor = Value
                    highlight.OutlineColor = Value
                end
            end
        end
    end
})

-- Corrigir o Toggle do Aimbot
AimbotTab:AddToggle({
    Name = "Aimbot",
    Default = false,
    Callback = function(Value)
        enableAimbot(Value) -- Chamar a função correta
    end
})

-- Na seção AimbotTab, adicione:
AimbotTab:AddBind({
    Name = "Bind Aimbot (Segurar)",
    Default = Enum.KeyCode.Q,
    Hold = true,
    Callback = function(value)
        aimbotHolding = value
    end
})

-- Corrigir os Sliders
AimbotTab:AddSlider({
    Name = "Aimbot Fov",
    Min = 50,
    Max = 500,
    Default = 100,
    Color = Color3.fromRGB(119, 18, 169),
    Increment = 1,
    ValueName = "FOV",
    Callback = function(Value)
        fovSize = Value
        updateFovCircle()
    end
})

AimbotTab:AddSlider({
    Name = "Aimbot Distance",
    Min = 50,
    Max = 500,
    Default = 100,
    Color = Color3.fromRGB(119, 18, 169),
    Increment = 1,
    ValueName = "Distance",
    Callback = function(Value)
        aimDistance = Value
    end
})



-- Na seção Aimbot (adicionar após os sliders existentes)
AimbotTab:AddColorpicker({
    Name = "Cor do FOV",
    Default = Color3.new(1, 1, 1),
    Callback = function(Value)
        fovColor = Value
        fovCircle.Color = Value
    end
})


-- Atualizar a função esp para usar as variáveis de cor
function esp(enabled)
    -- ... código existente ...
    
    local FillColor = chamFillColor
    local OutlineColor = chamOutlineColor
    
    -- ... restante do código ...
end

-- Atualizar a criação do FOV Circle para usar a variável de cor
fovCircle.Color = fovColorlocal 
