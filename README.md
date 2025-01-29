local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local Window = Fluent:CreateWindow({
    Title = "Macaco hub X prison life",
    SubTitle = "by Monaco",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local players = game:GetService("Players")
local tweenService = game:GetService("TweenService")

local player = players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local rootPart = character:WaitForChild("HumanoidRootPart")
local maxSpeed = 10000

-- Variáveis de controle para toggles
local KillAuraEnabled = false
local KillAllEnabled = false

-- Atualiza automaticamente o personagem e rootPart quando o jogador morrer
local function UpdateCharacterReferences()
    character = player.Character or player.CharacterAdded:Wait()
    rootPart = character:WaitForChild("HumanoidRootPart")
end

-- Inicializa as referências do personagem
UpdateCharacterReferences()

local door = workspace.Doors
local cellDoor = workspace.Prison_Cellblock.doors

-- Escuta quando o personagem é recriado
player.CharacterAdded:Connect(function()
    UpdateCharacterReferences()
end)

local DropdownPlayers = {}
for _, plr in pairs(players:GetPlayers()) do
    table.insert(DropdownPlayers, plr.Name)
end

-- Atualiza a lista de jogadores dinamicamente
players.PlayerAdded:Connect(function(plr)
    table.insert(DropdownPlayers, plr.Name)
end)

players.PlayerRemoving:Connect(function(plr)
    for i, name in ipairs(DropdownPlayers) do
        if name == plr.Name then
            table.remove(DropdownPlayers, i)
            break
        end
    end
end)

-- Guns
local Ak47 = workspace.Prison_ITEMS.giver:FindFirstChild("AK-47")
local Shotgun = workspace.Prison_ITEMS.giver:FindFirstChild("Remington 870")

-- Função para mover até um jogador
local function TweenToPlayer(targetName)
    local target = players:FindFirstChild(targetName)
    if not target or target == player then
        warn("Jogador inválido ou não encontrado!")
        return
    end

    local targetCharacter = target.Character
    if not targetCharacter then
        warn("Personagem do jogador não carregado!")
        return
    end

    local targetRoot = targetCharacter:FindFirstChild("HumanoidRootPart")
    if not targetRoot then
        warn("HumanoidRootPart do jogador não encontrado!")
        return
    end

    local distance = (targetRoot.Position - rootPart.Position).Magnitude
    local duration = distance / maxSpeed
    local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
    local tween = tweenService:Create(rootPart, tweenInfo, {CFrame = targetRoot.CFrame})

    tween:Play()
end

-- Função genérica para tween até um local
local function TweenToLocation(goal)
    if not goal then
        warn("Local de destino não encontrado!")
        return
    end

    local primaryPart = goal:FindFirstChild("CFrame") and goal or goal:FindFirstChildWhichIsA("BasePart")
    if not primaryPart then
        warn("Nenhuma parte utilizável encontrada no objeto!")
        return
    end

    local distance = (primaryPart.Position - rootPart.Position).Magnitude
    local duration = distance / maxSpeed
    local tweenInfo = TweenInfo.new(duration, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
    local tween = tweenService:Create(rootPart, tweenInfo, {CFrame = primaryPart.CFrame})

    tween:Play()
end

local function Kill()
    while KillAuraEnabled and wait() do
        for _, e in pairs(players:GetChildren()) do
            if e ~= player then
                local meleeEvent = game:GetService("ReplicatedStorage").meleeEvent
                meleeEvent:FireServer(e)
            end
        end
    end
end

local function KillAll()
    while KillAllEnabled and wait(0.1) do
        for _, v in pairs(players:GetChildren()) do
            pcall(function()
                if v ~= player and not v.Character:FindFirstChildOfClass("ForceField") and v.Character.Humanoid.Health > 0 then
                    player.Character.HumanoidRootPart.CFrame = v.Character.HumanoidRootPart.CFrame
                    game.ReplicatedStorage.meleeEvent:FireServer(v)
                end
            end)
        end
    end
end

-- Tabs e interface gráfica
local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = ""}),
    Teleport = Window:AddTab({ Title = "Teleport", Icon = "" }),
    Guns = Window:AddTab({ Title = "Guns", Icon = "" }),
    playerTab = Window:AddTab({ Title = "Player", Icon = "" }),
    Misc = Window:AddTab({ Title = "Misc", Icon = "" }),
}

local KillAuraToggle = Tabs.Main:AddToggle("MyToggle", {Title = "Kill aura", Default = false })

KillAuraToggle:OnChanged(function(state)
    KillAuraEnabled = state
    if state then
        task.spawn(Kill)
    end
end)

local KillAllToggle = Tabs.Main:AddToggle("MyToggle", {Title = "Kill All", Default = false })

KillAllToggle:OnChanged(function(state)
    KillAllEnabled = state
    if state then
        task.spawn(KillAll)
    end
end)

Tabs.Teleport:AddButton({
    Title = "Get Bandite",
    Callback = function()
        spawn(TweenToBandite)
    end
})

Tabs.Teleport:AddButton({
    Title = "Go to Prison",
    Callback = function()
        spawn(TweenToPrison)
    end
})

Tabs.Guns:AddButton({
    Title = "Get AK47",
    Callback = function()
        spawn(TweenToAk47)
    end
})

Tabs.Guns:AddButton({
    Title = "Get Shotgun",
    Callback = function()
        spawn(TweenToShotgun)
    end
})

local Dropdown = Tabs.Teleport:AddDropdown("Dropdown", {
    Title = "Teleport to player",
    Values = DropdownPlayers,
    Multi = false,
    Default = "Choose player",
    Callback = function(selected)
        spawn(TweenToPlayer)
    end
})

Tabs.Misc:AddButton({
    Title = "Delete all doors",
    Callback = function()
        if door and door.Parent then
            door:Destroy()
        else
            warn("As portas já foram destruídas ou não existem!")
        end
    end
})

Tabs.Misc:AddButton({
    Title = "Delete all cell doors",
    Callback = function()
        if cellDoor and cellDoor.Parent then
            cellDoor:Destroy()
        else
            warn("As portas já foram destruídas ou não existem!")
        end
    end
})

local SpeedPlayerSlider = Tabs.playerTab:AddSlider("Slider", {
    Title = "Player Speed",
    Description = "Change the player speed",
    Default = 1,
    Min = 1,
    Max = 10,
    Rounding = 1,
    Callback = function(Value)
        player.Character.Humanoid.WalkSpeed = 16 * Value
    else
        warn("Error to change the player speed")
    end
})
