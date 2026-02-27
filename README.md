-- ULTIMATE MOBILE HUB 2026  ALPHA (MAX MOBILE OPTIMIZED)
local success, Library = pcall(function()
    return loadstring(gameHttpGet(httpsraw.githubusercontent.comxHeptcKavo-UI-Librarymainsource.lua))()
end)

if not success then return end

local Window = Library.CreateLib(DRAKE Universal Hub, DarkTheme)

-- SERVIZI
local Players = gameGetService(Players)
local RunService = gameGetService(RunService)
local UserInputService = gameGetService(UserInputService)
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- SETTINGS
local AimSettings = { Enabled = false, Blatant = false, Smoothness = 5, Range = 400 }
local ESPConfig = { Box = false, Health = false, Snaplines = false }
local Movement = { Fly = false, Speed = 16, Jump = 50, Noclip = false }
local SelectedPlayer = nil

--------------------------------------------------
-- BOTÃO FLUTUANTE ARRASTÁVEL (ABRIR/FECHAR SEMPRE ATIVO)
--------------------------------------------------
local sg = Instance.new(ScreenGui, LocalPlayer.PlayerGui)
sg.Name = DRAKEFloatingButton
sg.ResetOnSpawn = false

local mainBtnFrame = Instance.new(Frame, sg)
mainBtnFrame.Size = UDim2.new(0, 65, 0, 65)
mainBtnFrame.Position = UDim2.new(0.05, 0, 0.4, 0)
mainBtnFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
mainBtnFrame.Active = true
mainBtnFrame.Draggable = true -- Trascinabile ovunque
Instance.new(UICorner, mainBtnFrame).CornerRadius = UDim.new(1, 0)

local toggleBtn = Instance.new(TextButton, mainBtnFrame)
toggleBtn.Size = UDim2.new(1, 0, 1, 0)
toggleBtn.BackgroundTransparency = 1
toggleBtn.Text = DRAKE
toggleBtn.TextColor3 = Color3.new(1, 1, 1)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextSize = 12
toggleBtn.MouseButton1ClickConnect(function() 
    LibraryToggleUI() 
end)

--------------------------------------------------
-- FAÇA O HUB ARRASTAR E ADICIONE A OPÇÃO DE MINIMIZAR (-)
--------------------------------------------------
task.spawn(function()
    local MainUI = LocalPlayer.PlayerGuiWaitForChild(DRAKE Universal Hub, 5) or gameGetService(CoreGui)WaitForChild(DRAKE  Universal Hub, 5)
    if MainUI then
        local MainFrame = MainUIFindFirstChild(Main)
        if MainFrame then
            -- Rendi l'Hub trascinabile (Mobile Fix)
            MainFrame.Active = true
            MainFrame.Draggable = true 

            -- Aggiunta tasto - (Minimize) accanto alla X
            local Header = MainFrameFindFirstChild(Header)
            if Header then
                local minBtn = Instance.new(TextButton, Header)
                minBtn.Name = MinimizeBtn
                minBtn.Size = UDim2.new(0, 30, 0, 30)
                minBtn.Position = UDim2.new(1, -65, 0, 5)
                minBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
                minBtn.Text = -
                minBtn.TextColor3 = Color3.new(1, 1, 1)
                minBtn.Font = Enum.Font.GothamBold
                minBtn.TextSize = 20
                Instance.new(UICorner, minBtn)

                local Container = MainFrameFindFirstChild(MainContainer) or MainFrameFindFirstChild(Container)
                local isMinimized = false
                
                minBtn.MouseButton1ClickConnect(function()
                    isMinimized = not isMinimized
                    if Container then Container.Visible = not isMinimized end
                    MainFrameTweenSize(isMinimized and UDim2.new(0, 525, 0, 40) or UDim2.new(0, 525, 0, 318), Out, Quad, 0.2, true)
                end)
            end
        end
    end
end)

--------------------------------------------------
-- PRO FLY ENGINE (CAMERA DIRECTIONAL)
--------------------------------------------------
local flyV, flyG
local function ToggleFly(v)
    Movement.Fly = v
    if not LocalPlayer.Character or not LocalPlayer.CharacterFindFirstChild(HumanoidRootPart) then return end
    local root = LocalPlayer.Character.HumanoidRootPart
    if v then
        flyV = Instance.new(BodyVelocity, root)
        flyV.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        flyG = Instance.new(BodyGyro, root)
        flyG.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        flyG.P = 3000
        task.spawn(function()
            while Movement.Fly do
                RunService.RenderSteppedWait()
                local hum = LocalPlayer.CharacterFindFirstChild(Humanoid)
                if hum then
                    flyG.CFrame = Camera.CFrame
                    if hum.MoveDirection.Magnitude  0 then
                        flyV.Velocity = Camera.CFrame.LookVector  (Movement.Speed  2)
                    else
                        flyV.Velocity = Vector3.zero
                    end
                end
            end
            if flyV then flyVDestroy() end
            if flyG then flyGDestroy() end
        end)
    else
        if flyV then flyVDestroy() end
        if flyG then flyGDestroy() end
    end
end

--------------------------------------------------
-- ESP & AIMBOT
--------------------------------------------------
local function GetClosest()
    local t, d = nil, AimSettings.Range
    for _, v in pairs(PlayersGetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.CharacterFindFirstChild(Head) and v.Character.Humanoid.Health  0 then
            local p, s = CameraWorldToViewportPoint(v.Character.Head.Position)
            if s then
                local m = (Vector2.new(p.X, p.Y) - Vector2.new(Camera.ViewportSize.X2, Camera.ViewportSize.Y2)).Magnitude
                if m  d then d = m t = v.Character.Head end
            end
        end
    end
    return t
end

RunService.RenderSteppedConnect(function()
    if AimSettings.Enabled then
        local target = GetClosest()
        if target then
            local cf = CFrame.new(Camera.CFrame.Position, target.Position)
            if AimSettings.Blatant then Camera.CFrame = cf
            else Camera.CFrame = Camera.CFrameLerp(cf, 1(AimSettings.Smoothness+1)) end
        end
    end
end)

local espTable = {}
local function CreateESP(plr)
    if plr == LocalPlayer then return end
    local d = { Box = Drawing.new(Square), HealthBarBack = Drawing.new(Line), HealthBar = Drawing.new(Line), Snap = Drawing.new(Line) }
    d.Box.Thickness = 2.5 d.Box.Filled = false d.HealthBarBack.Thickness = 5 d.HealthBarBack.Color = Color3.new(0,0,0) d.HealthBar.Thickness = 3 d.Snap.Thickness = 2.5 d.Snap.Color = Color3.new(1,1,1)
    espTable[plr] = d
end

RunService.RenderSteppedConnect(function()
    for plr, d in pairs(espTable) do
        if plr.Character and plr.CharacterFindFirstChild(HumanoidRootPart) and plr.Character.Humanoid.Health  0 then
            local hrp = plr.Character.HumanoidRootPart
            local pos, s = CameraWorldToViewportPoint(hrp.Position)
            if s then
                local h = math.abs(CameraWorldToViewportPoint(plr.Character.Head.Position + Vector3.new(0, 0.8, 0)).Y - CameraWorldToViewportPoint(hrp.Position - Vector3.new(0, 3.5, 0)).Y)
                local w = h  1.3
                d.Box.Visible = ESPConfig.Box d.Box.Size = Vector2.new(w, h) d.Box.Position = Vector2.new(pos.X - w2, pos.Y - h2)
                d.Box.Color = (plr.TeamColor == LocalPlayer.TeamColor) and Color3.new(0,1,0) or Color3.new(1,0,0)
                d.Snap.Visible = ESPConfig.Snaplines d.Snap.From = Vector2.new(Camera.ViewportSize.X2, Camera.ViewportSize.Y) d.Snap.To = Vector2.new(pos.X, pos.Y)
                d.HealthBarBack.Visible = ESPConfig.Health d.HealthBar.Visible = ESPConfig.Health
                if ESPConfig.Health then
                    local bx = (pos.X - w2) - 8 local hp = plr.Character.Humanoid.Health  plr.Character.Humanoid.MaxHealth
                    d.HealthBarBack.From = Vector2.new(bx, pos.Y + h2) d.HealthBarBack.To = Vector2.new(bx, pos.Y - h2)
                    d.HealthBar.From = Vector2.new(bx, pos.Y + h2) d.HealthBar.To = Vector2.new(bx, (pos.Y + h2) - (h  hp))
                    d.HealthBar.Color = Color3.new(1 - hp, hp, 0)
                end
            else for _, v in pairs(d) do v.Visible = false end end
        else for _, v in pairs(d) do v.Visible = false end end
    end
end)

Players.PlayerAddedConnect(CreateESP)
for _, p in pairs(PlayersGetPlayers()) do CreateESP(p) end

--------------------------------------------------
-- TABS & INPUTS
--------------------------------------------------
local TabCombat = WindowNewTab(Combat)
local SecA = TabCombatNewSection(Aimbot)
SecANewToggle(Enable Aimbot, Lock on, function(v) AimSettings.Enabled = v end)
SecANewToggle(Blatant, Instant, function(v) AimSettings.Blatant = v end)
SecANewTextBox(Smoothness (0-50), Lower = Faster, function(t) AimSettings.Smoothness = tonumber(t) or 5 end)

local TabMove = WindowNewTab(Movement)
local SecM = TabMoveNewSection(Pro Cheats)
SecMNewToggle(Fly, Camera-Directional, function(v) ToggleFly(v) end)
SecMNewToggle(Noclip, No Walls, function(v) Movement.Noclip = v end)
SecMNewTextBox(Speed, Walk & Fly, function(t) Movement.Speed = tonumber(t) or 16 end)
SecMNewTextBox(JumpPower, Jump height, function(t) Movement.Jump = tonumber(t) or 50 end)

local TabVisual = WindowNewTab(Visual)
local SecV = TabVisualNewSection(ESP Settings)
SecVNewToggle(Box ESP, Full Body, function(v) ESPConfig.Box = v end)
SecVNewToggle(Snaplines, Thick, function(v) ESPConfig.Snaplines = v end)
SecVNewToggle(Health Bar, Side Only, function(v) ESPConfig.Health = v end)

local TabPlayer = WindowNewTab(Players)
local SecP = TabPlayerNewSection(TP Utils)
local PlayerDropdown = SecPNewDropdown(Select Player, None, {}, function(v) SelectedPlayer = PlayersFindFirstChild(v) end)
SecPNewButton(Refresh, Update List, function()
    local names = {}
    for _, p in pairs(PlayersGetPlayers()) do if p ~= LocalPlayer then table.insert(names, p.Name) end end
    PlayerDropdownRefresh(names)
end)
SecPNewButton(Teleport To, TP, function()
    if SelectedPlayer and SelectedPlayer.Character then LocalPlayer.Character.HumanoidRootPart.CFrame = SelectedPlayer.Character.HumanoidRootPart.CFrame end
end)

RunService.SteppedConnect(function()
    if LocalPlayer.Character and LocalPlayer.CharacterFindFirstChild(Humanoid) then
        if Movement.Noclip then
            for _, v in pairs(LocalPlayer.CharacterGetDescendants()) do if vIsA(BasePart) then v.CanCollide = false end end
        end
        LocalPlayer.Character.Humanoid.WalkSpeed = Movement.Speed
        LocalPlayer.Character.Humanoid.JumpPower = Movement.Jump
    end
end)

LibraryNotify(DRAKE V4.3 - DRAKE HUB READY, 5)
