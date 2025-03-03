local OrionLib = loadstring(game:HttpGet(('https://raw.githubusercontent.com/jensonhirst/Orion/main/source')))()

local Window = OrionLib:MakeWindow({Name = "Japa Menu V3", HidePremium = false, SaveConfig = true, ConfigFolder = "OrionTest"})

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
local moveSpeed = 0.5
local rotationSpeed = 0.001
local rotationSpeedQ = 0.001

local function onMouseMove(input)
    if freecamEnabled and input.UserInputType == Enum.UserInputType.MouseMovement then
        mouseDelta = input.Delta
    end
end

local function enableFreecam()
    if not freecamEnabled then
        freecamEnabled = true
        Camera.CameraType = Enum.CameraType.Scriptable
        Camera.CFrame = humanoidRootPart.CFrame * CFrame.new(cameraOffset)
        
        -- Desabilitar movimentação do personagem
        if humanoid then
            humanoid.WalkSpeed = 0
            humanoid.JumpPower = 0
            humanoid.PlatformStand = true -- Evita que o personagem se mova ou pule
        end
        
        -- Bloqueia a entrada das teclas de movimento (W, A, S, D)
        UserInputService.InputBegan:Connect(function(input, gameProcessedEvent)
            if freecamEnabled and not gameProcessedEvent then
                if input.UserInputType == Enum.UserInputType.Keyboard then
                    if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.A or input.KeyCode == Enum.KeyCode.S or input.KeyCode == Enum.KeyCode.D then
                        input:Stop() -- Bloqueia as teclas de movimento
                    end
                end
            end
        end)

        mouseMovementConnection = UserInputService.InputChanged:Connect(onMouseMove)
        
        RunService.RenderStepped:Connect(function()
            if freecamEnabled then
                local moveDirection = Vector3.new(0, 0, 0)
                -- Movimenta a câmera com W, A, S, D
                if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDirection = moveDirection + Vector3.new(0, 0, -1) end
                if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDirection = moveDirection + Vector3.new(0, 0, 1) end
                if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDirection = moveDirection + Vector3.new(-1, 0, 0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDirection = moveDirection + Vector3.new(1, 0, 0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDirection = moveDirection + Vector3.new(0, -1, 0) end
                
                Camera.CFrame = Camera.CFrame * CFrame.new(moveDirection * moveSpeed)
                
                -- Movimentação da câmera para rotação com Q e E
                if UserInputService:IsKeyDown(Enum.KeyCode.Q) then
                    Camera.CFrame = Camera.CFrame * CFrame.Angles(0, rotationSpeedQ, 0)
                end
                if UserInputService:IsKeyDown(Enum.KeyCode.E) then
                    Camera.CFrame = Camera.CFrame * CFrame.Angles(0, -rotationSpeedQ, 0)
                end
                
                -- Controle da rotação com o mouse
                if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
                    local sensitivity = 0.1
                    local yaw = CFrame.Angles(0, -mouseDelta.X * sensitivity, 0)
                    local pitch = CFrame.Angles(-mouseDelta.Y * sensitivity, 0, 0)
                    Camera.CFrame = Camera.CFrame * yaw * pitch
                    mouseDelta = Vector2.new(0, 0)
                end
            end
        end)
    end
end

local function disableFreecam()
    freecamEnabled = false
    Camera.CameraType = Enum.CameraType.Custom
    Camera.CFrame = humanoidRootPart.CFrame
    if mouseMovementConnection then
        mouseMovementConnection:Disconnect()
    end
    if humanoid then
        humanoid.WalkSpeed = 16
        humanoid.JumpPower = 50
        humanoid.PlatformStand = false -- Restaura o movimento normal do personagem
    end
end


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
    Name = "Self",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})


OrionLib:MakeNotification({
    Name = "Japa Menu V3",
    Content = "Menu injetado Com Sucesso!",
    Image = "rbxassetid://4483345998",
    Time = 5
})

local WallTab = Window:MakeTab({
    Name = "Visuals",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local Section = WallTab:AddSection({
    Name = "Esp"
})

local AimbotTab = Window:MakeTab({
    Name = "Aimbot",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local Section = AimbotTab:AddSection({
    Name = "Aimbot"
})

local Section = Tab:AddSection({
    Name = "Self"
})

local ConfigTab = Window:MakeTab({
    Name = "Misc",
    Icon = "rbxassetid://4483345998",
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
    Min = 1,
    Max = 5,
    Default = 1,
    Increment = 1,
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

-- Na seção AimbotTab, adicione:
AimbotTab:AddBind({
    Name = "Bind Aimbot (Segurar)",
    Default = Enum.KeyCode.Q,
    Hold = true,
    Callback = function(value)
        aimbotHolding = value
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
