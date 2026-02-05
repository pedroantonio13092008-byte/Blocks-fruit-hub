--[[
    ================================================================================
    ||                                                                            ||
    ||                              NautilusHub v1.0                              ||
    ||                                                                            ||
    ||      Script otimizado e montado por Manus, a pedido do usuário.            ||
    ||      Foco em desempenho, velocidade de farm e estabilidade.                ||
    ||                                                                            ||
    ================================================================================
]]

-- ================================================================================
-- ||                        SERVIÇOS E VARIÁVEIS GLOBAIS                        ||
-- ================================================================================

-- Carregando serviços do Roblox de forma segura
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local VirtualUser = game:GetService("VirtualUser")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local TeleportService = game:GetService("TeleportService")

local LocalPlayer = Players.LocalPlayer
local MyPlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- Variáveis de estado do mundo para identificar o "Sea" atual
local World1, World2, World3 = false, false, false
local currentPlaceId = game.PlaceId

if currentPlaceId == 2753915549 then
    World1 = true
elseif currentPlaceId == 4442272183 then
    World2 = true
elseif currentPlaceId == 7449423635 then
    World3 = true
else
    LocalPlayer:Kick("NautilusHub: Este mapa não é suportado.")
end

-- Configurações globais (getgenv para acesso universal no script)
getgenv().Settings = {
    AutoFarm = false,
    FastAttack = true,
    FastAttackDelay = 0.1,
    SelectedWeapon = "Melee",
    BringMobs = true,
    BringMobsRange = 350,
    UseSkills = { Z = true, X = true, C = true, V = true, F = true },
    -- Adicione outras configurações padrão aqui
}

-- ================================================================================
-- ||                           FUNÇÕES UTILITÁRIAS                            ||
-- ================================================================================

-- Função para arredondar números
local function round(n)
    return math.floor(tonumber(n) + 0.5)
end

-- Função para teleporte otimizado e mais seguro
local function teleport(targetCFrame)
    if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    
    local rootPart = LocalPlayer.Character.HumanoidRootPart
    local distance = (rootPart.Position - targetCFrame.Position).Magnitude

    -- Teleporte instantâneo para distâncias curtas, tween rápido para longas (simula dash)
    if distance < 150 then
        rootPart.CFrame = targetCFrame
        return
    end
    
    local speed = 800 -- Velocidade base aumentada para ser mais rápido
    if distance < 500 then speed = 1200 end
    if distance > 2000 then speed = 600 end

    local tweenInfo = TweenInfo.new(distance / speed, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(rootPart, tweenInfo, { CFrame = targetCFrame })
    tween:Play()
    return tween
end

-- Função para equipar arma de forma confiável
local function equipWeapon(weaponName)
    if not weaponName then return end
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild(weaponName) then
        return -- Arma já está equipada
    end
    if LocalPlayer.Backpack:FindFirstChild(weaponName) then
        LocalPlayer.Character.Humanoid:EquipTool(LocalPlayer.Backpack[weaponName])
    end
end

-- Função para auto ativação do Haki de Armamento (Buso)
local function autoHaki()
    if not LocalPlayer.Character:FindFirstChild("HasBuso") then
        ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
    end
end

-- ================================================================================
-- ||                           LÓGICA DE COMBATE E FARM                         ||
-- ================================================================================

-- Otimização de ataque rápido (Fast Attack)
pcall(function()
    local CombatFramework = require(LocalPlayer.PlayerScripts.CombatFramework)
    local CombatUtil = debug.getupvalues(CombatFramework)[2]

    RunService.RenderStepped:Connect(function()
        if getgenv().Settings.FastAttack then
            if typeof(CombatUtil) == "table" and CombatUtil.activeController then
                CombatUtil.activeController.timeToNextAttack = 0
                CombatUtil.activeController.hitboxMagnitude = 70 -- Aumenta o alcance do hit
                CombatUtil.activeController.attacking = false
                CombatUtil.activeController.blocking = false
            end
        end
    end)
end)

-- Função principal de quests (Otimizada e centralizada)
function CheckQuest()
    if not LocalPlayer or not LocalPlayer:FindFirstChild("Data") then return {} end
    local myLevel = LocalPlayer.Data.Level.Value
    
    local questData = {}

    -- A lógica "if myLevel >= X and myLevel <= Y" é mais segura e clara
    if World1 then
        if myLevel >= 1 and myLevel <= 9 then
            questData = { Mon = "Bandit", LevelQuest = 1, NameQuest = "BanditQuest1", CFrameQuest = CFrame.new(1059.37, 15.44, 1550.42), CFrameMon = CFrame.new(1045.96, 27.00, 1560.82) }
        elseif myLevel >= 10 and myLevel <= 14 then
            questData = { Mon = "Monkey", LevelQuest = 1, NameQuest = "JungleQuest", CFrameQuest = CFrame.new(-1598.08, 35.55, 153.37), CFrameMon = CFrame.new(-1448.51, 67.85, 11.46) }
        -- ... (COLE O RESTANTE DAS SUAS CONDIÇÕES DE QUEST AQUI, SEGUINDO O MESMO PADRÃO)
        elseif myLevel >= 650 then
            questData = { Mon = "Galley Captain", LevelQuest = 2, NameQuest = "FountainQuest", CFrameQuest = CFrame.new(5259.81, 37.35, 4050.02), CFrameMon = CFrame.new(5441.95, 42.50, 4950.09) }
        end
    elseif World2 then
        if myLevel >= 700 and myLevel <= 724 then
            questData = { Mon = "Raider", LevelQuest = 1, NameQuest = "Area1Quest", CFrameQuest = CFrame.new(-429.54, 71.76, 1836.18), CFrameMon = CFrame.new(-728.32, 52.77, 2345.77) }
        -- ... (COLE O RESTANTE DAS SUAS CONDIÇÕES DE QUEST AQUI)
        elseif myLevel >= 1450 then
             questData = { Mon = "Water Fighter", LevelQuest = 2, NameQuest = "ForgottenQuest", CFrameQuest = CFrame.new(-3054.44, 235.54, -10142.81), CFrameMon = CFrame.new(-3352.90, 285.01, -10534.84) }
        end
    elseif World3 then
        if myLevel >= 1500 and myLevel <= 1524 then
            questData = { Mon = "Pirate Millionaire", LevelQuest = 1, NameQuest = "PiratePortQuest", CFrameQuest = CFrame.new(-290.07, 42.90, 5581.58), CFrameMon = CFrame.new(-245.99, 47.30, 5584.10) }
        -- ... (COLE O RESTANTE DAS SUAS CONDIÇÕES DE QUEST AQUI)
        elseif myLevel >= 2525 then
            questData = { Mon = "Isle Champion", LevelQuest = 2, NameQuest = "TikiQuest2", CFrameQuest = CFrame.new(-16539.07, 55.68, 1051.57), CFrameMon = CFrame.new(-16933.21, 93.35, 999.45) }
        end
    end
    
    return questData
end

-- Loop principal de AutoFarm
task.spawn(function()
    while true do
        task.wait() -- Previne que o loop trave se o AutoFarm for desativado
        if getgenv().Settings.AutoFarm then
            local currentQuest = CheckQuest()
            if not currentQuest or not currentQuest.Mon then
                print("NautilusHub: Nenhuma quest encontrada para o nível atual. Aguardando...")
                task.wait(5)
            else
                local questGUI = MyPlayerGui.Main.Quest
                -- 1. PEGAR A QUEST
                if not questGUI.Visible or not string.find(questGUI.Container.QuestTitle.Title.Text, currentQuest.Mon) then
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("AbandonQuest")
                    teleport(currentQuest.CFrameQuest)
                    task.wait(0.5) -- Pequena pausa para simular interação humana
                    ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", currentQuest.NameQuest, currentQuest.LevelQuest)
                    task.wait(0.5)
                end

                -- 2. ENCONTRAR E ATACAR O MONSTRO
                local targetMob, closestDist = nil, math.huge
                for _, mob in pairs(Workspace.Enemies:GetChildren()) do
                    if mob.Name == currentQuest.Mon and mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                        local dist = (LocalPlayer.Character.HumanoidRootPart.Position - mob.HumanoidRootPart.Position).Magnitude
                        if dist < closestDist then
                            targetMob = mob
                            closestDist = dist
                        end
                    end
                end

                if targetMob then
                    local mobRoot = targetMob.HumanoidRootPart
                    local mobHumanoid = targetMob.Humanoid
                    
                    -- Puxar todos os monstros próximos do mesmo tipo (Bring Mobs)
                    if getgenv().Settings.BringMobs then
                        for _, mob in pairs(Workspace.Enemies:GetChildren()) do
                            if mob.Name == currentQuest.Mon and mob:FindFirstChild("HumanoidRootPart") then
                                if (mob.HumanoidRootPart.Position - mobRoot.Position).Magnitude < getgenv().Settings.BringMobsRange then
                                    mob.HumanoidRootPart.CFrame = mobRoot.CFrame
                                    mob:FindFirstChildOfClass("Humanoid"):ChangeState(14) -- Estado Ragdoll para imobilizar
                                end
                            end
                        end
                    end

                    -- Atacar
                    equipWeapon(getgenv().Settings.SelectedWeapon)
                    autoHaki()
                    
                    local attackPosition = mobRoot.CFrame * CFrame.new(0, 0, -5) -- Posição segura para atacar
                    teleport(attackPosition)
                    
                    while targetMob and targetMob.Parent and mobHumanoid.Health > 0 and getgenv().Settings.AutoFarm do
                        VirtualUser:Button1Down(Vector2.new(1280, 672)) -- Simula o clique
                        task.wait(getgenv().Settings.FastAttackDelay)
                    end
                    VirtualUser:Button1Up(Vector2.new(1280, 672))

                else
                    -- Se não houver monstros, ir para o local de spawn
                    teleport(currentQuest.CFrameMon)
                    task.wait(1) -- Espera os monstros renascerem
                end
            end
        end
    end
end)


-- ================================================================================
-- ||                           INTERFACE GRÁFICA (OrionLib)                     ||
-- ================================================================================
-- Esta é a estrutura para a sua interface. Cole o seu código completo da OrionLib aqui dentro.
-- O exemplo abaixo mostra como integrar as novas configurações.

task.spawn(function()
    -- Tenta carregar a biblioteca de um arquivo local para ser mais rápido. Se não existir, baixa.
    if not isfile("OrionLib.lua") then
        local source = game:HttpGet("https://raw.githubusercontent.com/shlexware/Orion/main/source")
        writefile("OrionLib.lua", source)
    end
    
    local OrionLib = loadstring(readfile("OrionLib.lua"))()
    local Window = OrionLib:MakeWindow({Name = "NautilusHub", HidePremium = false, SaveConfig = true, ConfigFolder = "NautilusHub_Config"})

    -- Aba Principal
    local MainTab = Window:MakeTab({
        Name = "Principal",
        Icon = "rbxassetid://119980140458596",
    })

    local FarmSection = MainTab:AddSection({ Name = "Auto Farm" })

    FarmSection:AddToggle({
        Name = "Ativar Auto Farm",
        Default = getgenv().Settings.AutoFarm,
        Callback = function(value)
            getgenv().Settings.AutoFarm = value
            if not value then
                VirtualUser:Button1Up(Vector2.new(1280, 672)) -- Garante que o clique pare ao desativar
            end
        end
    }):SetFlag("AutoFarmToggle") -- Adiciona uma flag para salvar/carregar

    FarmSection:AddDropdown({
        Name = "Arma Principal",
        Default = getgenv().Settings.SelectedWeapon,
        Options = {"Melee", "Sword", "Fruit", "Gun"},
        Callback = function(value)
            getgenv().Settings.SelectedWeapon = value
        end
    }):SetFlag("SelectedWeaponDropdown")

    -- Aba de Configurações
    local SettingsTab = Window:MakeTab({
        Name = "Configurações",
        Icon = "rbxassetid://119980140458596",
    })

    local CombatSection = SettingsTab:AddSection({ Name = "Combate" })

    CombatSection:AddToggle({
        Name = "Ataque Rápido",
        Default = getgenv().Settings.FastAttack,
        Callback = function(value)
            getgenv().Settings.FastAttack = value
        end
    }):SetFlag("FastAttackToggle")

    CombatSection:AddSlider({
        Name = "Velocidade do Ataque",
        Min = 0.05,
        Max = 0.5,
        Default = getgenv().Settings.FastAttackDelay,
        Increment = 0.05,
        ValueName = "s",
        Callback = function(value)
            getgenv().Settings.FastAttackDelay = value
        end
    }):SetFlag("FastAttackDelaySlider")

    CombatSection:AddToggle({
        Name = "Puxar Monstros (Bring Mobs)",
        Default = getgenv().Settings.BringMobs,
        Callback = function(value)
            getgenv().Settings.BringMobs = value
        end
    }):SetFlag("BringMobsToggle")
    
    -- ================================================================================
    -- ||                  COLE O RESTO DO SEU CÓDIGO DA ORIONLIB AQUI               ||
    -- ||      (ESPs, Teleportes, Quests, etc., seguindo o mesmo padrão acima)       ||
    -- ================================================================================
    
    
    
    OrionLib:Init()
end)

-- Mensagem final de confirmação no console do desenvolvedor
print("NautilusHub v1.0 carregado com sucesso! Otimizado por Manus.")
