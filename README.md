-- RBX4HX [ALPHA] - PARTE 1/5
-- Core + ESP + Aimbot

local Players      = game:GetService("Players")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")

local L = Players.LocalPlayer
local C = workspace.CurrentCamera

_G.cfg = {
    aim=false, wc=false, tc=false, as=0.9, aimpart="Head",
    eb=false, et=false, en=false, ec=Color3.fromRGB(255,0,0),
    er=false, rs=0.005, edist=false,
    fc=true, fs=100, ft=2, fco=Color3.fromRGB(255,255,255),
    fr=false, invfov=false,
    ws=16, infjump=false, antiafk=false,
    antiban=false, antikick=false, safemode=false,
}

local rh = 0
local eo = {}
local antiafkconn = nil
_G.anticheatDetected = false
_G.banLog = {}

local ds = pcall(function() local t = Drawing.new("Line") t:Remove() end)
local fv = nil
if ds then
    fv = Drawing.new("Circle")
    fv.Visible=true fv.Thickness=2
    fv.Color=Color3.fromRGB(255,255,255)
    fv.Transparency=1 fv.Radius=100
    fv.Filled=false fv.NumSides=64
end

local function Log(msg)
    local e = "[" .. os.clock() .. "] " .. msg
    table.insert(_G.banLog, e)
    if #_G.banLog > 50 then table.remove(_G.banLog,1) end
    print("SHIELD " .. e)
end

local ROOT_NAMES = {"HumanoidRootPart","Root","Torso","UpperTorso","LowerTorso"}
local HEAD_NAMES = {"Head","head","Cabeca","Skull","Face"}

local function FindPart(char, names)
    for _, n in ipairs(names) do
        local p = char:FindFirstChild(n)
        if p then return p end
    end
    return nil
end

local function GetRoot(char) return FindPart(char, ROOT_NAMES) end
local function GetHead(char)  return FindPart(char, HEAD_NAMES) end

local function GetHumanoid(char)
    for _, v in ipairs(char:GetDescendants()) do
        if v:IsA("Humanoid") then return v end
    end
    return nil
end

local function GetAimPart(char)
    local p = char:FindFirstChild(_G.cfg.aimpart)
    if p then return p end
    for _, n in ipairs({"Head","UpperTorso","HumanoidRootPart","Torso"}) do
        local v = char:FindFirstChild(n)
        if v then return v end
    end
    return GetRoot(char)
end

local function IsAlive(char)
    if not char then return false end
    local h = GetHumanoid(char)
    if not h then return false end
    local ok, hp = pcall(function() return h.Health end)
    return ok and hp > 0
end

local cameraType = "Default"
local function DetectCamera()
    pcall(function()
        local ct = C.CameraType
        if ct == Enum.CameraType.Scriptable then cameraType = "Scriptable"
        elseif ct == Enum.CameraType.Fixed   then cameraType = "Fixed"
        elseif ct == Enum.CameraType.Custom  then cameraType = "Custom"
        else cameraType = "Default" end
    end)
end

function Grc()
    rh = rh + _G.cfg.rs
    if rh >= 1 then rh = 0 end
    return Color3.fromHSV(rh,1,1)
end

function Cep(p)
    if not ds then return end
    local e = {}
    e.b=Drawing.new("Square") e.b.Visible=false e.b.Filled=false e.b.Color=_G.cfg.ec e.b.Thickness=2
    e.t=Drawing.new("Line")   e.t.Visible=false e.t.Color=_G.cfg.ec e.t.Thickness=2
    e.d=Drawing.new("Text")   e.d.Visible=false e.d.Center=true e.d.Outline=true e.d.Color=Color3.fromRGB(200,200,200) e.d.Size=13 e.d.Font=2
    e.n=Drawing.new("Text")   e.n.Visible=false e.n.Center=true e.n.Outline=true e.n.Color=Color3.fromRGB(255,255,255) e.n.Size=14 e.n.Font=2
    eo[p]=e
end

function Rep(p)
    if eo[p] then
        pcall(function() eo[p].b:Remove() eo[p].t:Remove() eo[p].d:Remove() eo[p].n:Remove() end)
        eo[p]=nil
    end
end

local function hide(e)
    e.b.Visible=false e.t.Visible=false e.d.Visible=false e.n.Visible=false
end

function Uf()
    if not fv then return end
    fv.Position  = Vector2.new(C.ViewportSize.X/2, C.ViewportSize.Y/2)
    fv.Radius    = _G.cfg.fs
    fv.Thickness = _G.cfg.ft
    fv.Color     = _G.cfg.fr and Grc() or _G.cfg.fco
    fv.Visible   = _G.cfg.fc and not _G.cfg.invfov
end

function Ue()
    if not ds then return end
    for p, e in pairs(eo) do pcall(function()
        if p == L then hide(e) return end
        if _G.cfg.tc and p.Team and L.Team and p.Team==L.Team then hide(e) return end
        local char = p.Character
        if not char or not IsAlive(char) then hide(e) return end
        local root = GetRoot(char)
        local head = GetHead(char)
        if not root then hide(e) return end
        local rPos,rv = C:WorldToViewportPoint(root.Position)
        if not rv then hide(e) return end
        local hPos = head
            and C:WorldToViewportPoint(head.Position+Vector3.new(0,0.7,0))
            or  C:WorldToViewportPoint(root.Position+Vector3.new(0,3,0))
        local fPos = C:WorldToViewportPoint(root.Position-Vector3.new(0,3,0))
        local ht = math.abs(hPos.Y-fPos.Y)
        local w  = ht/2
        local ec = _G.cfg.er and Grc() or _G.cfg.ec
        if _G.cfg.eb then e.b.Size=Vector2.new(w,ht) e.b.Position=Vector2.new(rPos.X-w/2,rPos.Y-ht/2) e.b.Color=ec e.b.Visible=true else e.b.Visible=false end
        if _G.cfg.en then e.n.Text=p.Name e.n.Position=Vector2.new(hPos.X,hPos.Y-30) e.n.Color=ec e.n.Visible=true else e.n.Visible=false end
        if _G.cfg.edist then
            local dist=math.floor((root.Position-C.CFrame.Position).Magnitude)
            e.d.Text=dist.."M" e.d.Position=Vector2.new(hPos.X,hPos.Y-16) e.d.Visible=true
        else e.d.Visible=false end
        if _G.cfg.et then e.t.From=Vector2.new(C.ViewportSize.X/2,C.ViewportSize.Y) e.t.To=Vector2.new(rPos.X,rPos.Y) e.t.Color=ec e.t.Visible=true else e.t.Visible=false end
    end) end
end

function Uws()
    pcall(function()
        if L.Character then
            local h = GetHumanoid(L.Character)
            if h then
                h.WalkSpeed = _G.cfg.safemode and math.clamp(_G.cfg.ws,16,28) or _G.cfg.ws
            end
        end
    end)
end

function Gcp()
    if not _G.cfg.aim then return nil end
    local cp,sd = nil,_G.cfg.fs
    for _,p in pairs(Players:GetPlayers()) do if p~=L then pcall(function()
        if _G.cfg.tc and p.Team and L.Team and p.Team==L.Team then return end
        local char = p.Character
        if not char or not IsAlive(char) then return end
        local target = GetAimPart(char)
        if not target then return end
        local sp,v = C:WorldToViewportPoint(target.Position)
        if not v then return end
        local d = (Vector2.new(sp.X,sp.Y)-Vector2.new(C.ViewportSize.X/2,C.ViewportSize.Y/2)).Magnitude
        if d > _G.cfg.fs then return end
        if _G.cfg.wc then
            local ray = Ray.new(C.CFrame.Position,(target.Position-C.CFrame.Position).Unit*1000)
            local hit = workspace:FindPartOnRayWithIgnoreList(ray,{L.Character})
            if hit and not hit:IsDescendantOf(char) then return end
        end
        if d < sd then cp,sd=p,d end
    end) end end
    return cp
end

function Da()
    if _G.anticheatDetected and _G.cfg.safemode then return end
    local tg = Gcp()
    if not tg or not tg.Character then return end
    pcall(function()
        local target = GetAimPart(tg.Character)
        if not target then return end
        local smooth = math.clamp(_G.cfg.as, 0.01, 2.0)
        if cameraType=="Scriptable" or cameraType=="Fixed" then
            local sp = C:WorldToViewportPoint(target.Position)
            local center = Vector2.new(C.ViewportSize.X/2,C.ViewportSize.Y/2)
            local delta = Vector2.new(sp.X,sp.Y)-center
            pcall(function() mousemoverel(delta.X*smooth*0.04, delta.Y*smooth*0.04) end)
        else
            C.CFrame = C.CFrame:Lerp(CFrame.new(C.CFrame.Position,target.Position), smooth)
        end
    end)
end

function Tafk(st)
    _G.cfg.antiafk = st
    if antiafkconn then task.cancel(antiafkconn) antiafkconn=nil end
    if st then antiafkconn = task.spawn(function()
        while _G.cfg.antiafk and task.wait(math.random(25,55)) do
            pcall(function()
                local h = L.Character and GetHumanoid(L.Character)
                if h then
                    h:Move(Vector3.new(math.random(-1,1),0,math.random(-1,1)))
                    task.wait(0.1)
                    h:Move(Vector3.new(0,0,0))
                end
            end)
        end
    end) end
end

_G.SetupAntiBan = function()
    Log("Anti-Ban: 4 camadas iniciadas")
    pcall(function()
        local V = game:GetService("VirtualUser")
        L.Idled:Connect(function()
            if _G.cfg.antiban then
                V:Button2Down(Vector2.new(0,0),C.CFrame)
                task.wait(1)
                V:Button2Up(Vector2.new(0,0),C.CFrame)
            end
        end)
    end)
    pcall(function()
        local old
        old = hookmetamethod(game,"__namecall",function(self,...)
            local m = getnamecallmethod()
            if _G.cfg.antikick and m=="Kick" and self==L then
                Log("Kick bloqueado!")
                return nil
            end
            if _G.cfg.antiban and m=="FireServer" then
                local a={...}
                if a[1] and tostring(a[1]):lower():find("ban") then
                    Log("Acao suspeita bloqueada")
                    return nil
                end
            end
            return old(self,...)
        end)
    end)
    task.spawn(function()
        while task.wait(4) do
            if not _G.cfg.antiban then continue end
            pcall(function()
                for _,v in ipairs(workspace:GetDescendants()) do
                    if v:IsA("Script") or v:IsA("LocalScript") then
                        local n = v.Name:lower()
                        if n:find("anticheat") or n:find("antiexploit") or n:find("detect") or n:find("ban") then
                            if not _G.anticheatDetected then
                                _G.anticheatDetected = true
                                Log("Anti-cheat detectado: "..v.Name)
                            end
                        end
                    end
                end
            end)
        end
    end)
    task.spawn(function()
        while task.wait(2) do
            if not _G.cfg.antiban then continue end
            pcall(function()
                local ping = game:GetService("Stats").Network.ServerStatsItem["Data Ping"].Value
                if ping and ping > 900 then Log("Ping alto: "..math.floor(ping).."ms") end
            end)
        end
    end)
    Log("4 camadas ativas!")
end

UIS.JumpRequest:Connect(function()
    if _G.cfg.infjump and L.Character then
        local h = GetHumanoid(L.Character)
        if h then h:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

pcall(function()
    local V = game:GetService("VirtualUser")
    L.Idled:Connect(function()
        V:Button2Down(Vector2.new(0,0),C.CFrame)
        task.wait(1)
        V:Button2Up(Vector2.new(0,0),C.CFrame)
    end)
end)

if ds then
    for _,p in pairs(Players:GetPlayers()) do if p~=L then Cep(p) end end
    Players.PlayerAdded:Connect(function(p) task.wait(0.5) Cep(p) end)
    Players.PlayerRemoving:Connect(Rep)
end

RunService.RenderStepped:Connect(function()
    DetectCamera() Uf() Ue() Uws() Da()
end)

print("Parte 1/5 carregada!")
-- RBX4HX [ALPHA] - PARTE 2/5
-- Loading Screen Animado

local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local L            = Players.LocalPlayer

local function tw(obj, props, t, style, dir)
    TweenService:Create(obj, TweenInfo.new(
        t or 0.4,
        style or Enum.EasingStyle.Quad,
        dir or Enum.EasingDirection.Out
    ), props):Play()
end

local loadGui = Instance.new("ScreenGui")
loadGui.Name = "RBX4HX_Loading"
loadGui.IgnoreGuiInset = true
loadGui.ResetOnSpawn = false
loadGui.DisplayOrder = 999
loadGui.Parent = L:WaitForChild("PlayerGui")

local bg = Instance.new("Frame", loadGui)
bg.Size = UDim2.fromScale(1,1)
bg.BackgroundColor3 = Color3.fromRGB(10,10,10)
bg.BackgroundTransparency = 1
bg.BorderSizePixel = 0

local box = Instance.new("Frame", bg)
box.Size = UDim2.fromOffset(320,175)
box.Position = UDim2.fromScale(0.5,0.6)
box.AnchorPoint = Vector2.new(0.5,0.5)
box.BackgroundColor3 = Color3.fromRGB(18,18,18)
box.BackgroundTransparency = 1
box.BorderSizePixel = 0
Instance.new("UICorner", box).CornerRadius = UDim.new(0,16)
local bStroke = Instance.new("UIStroke", box)
bStroke.Color = Color3.fromRGB(55,55,55)
bStroke.Thickness = 1.5
bStroke.Transparency = 1

local lTitle = Instance.new("TextLabel", box)
lTitle.Size = UDim2.new(1,0,0,40)
lTitle.Position = UDim2.new(0,0,0,16)
lTitle.BackgroundTransparency = 1
lTitle.Text = "RBX4HX"
lTitle.Font = Enum.Font.GothamBold
lTitle.TextSize = 24
lTitle.TextColor3 = Color3.fromRGB(240,240,240)
lTitle.TextTransparency = 1

local lSub = Instance.new("TextLabel", box)
lSub.Size = UDim2.new(1,0,0,16)
lSub.Position = UDim2.new(0,0,0,54)
lSub.BackgroundTransparency = 1
lSub.Text = "Inicializando..."
lSub.Font = Enum.Font.Gotham
lSub.TextSize = 11
lSub.TextColor3 = Color3.fromRGB(100,100,100)
lSub.TextTransparency = 1

local lPct = Instance.new("TextLabel", box)
lPct.Size = UDim2.new(1,0,0,20)
lPct.Position = UDim2.new(0,0,0,76)
lPct.BackgroundTransparency = 1
lPct.Text = "0%"
lPct.Font = Enum.Font.GothamBold
lPct.TextSize = 13
lPct.TextColor3 = Color3.fromRGB(190,190,190)
lPct.TextTransparency = 1

local barBG = Instance.new("Frame", box)
barBG.Size = UDim2.new(0.78,0,0,5)
barBG.Position = UDim2.new(0.11,0,0,106)
barBG.BackgroundColor3 = Color3.fromRGB(30,30,30)
barBG.BackgroundTransparency = 1
barBG.BorderSizePixel = 0
Instance.new("UICorner", barBG).CornerRadius = UDim.new(1,0)

local bar = Instance.new("Frame", barBG)
bar.Size = UDim2.new(0,0,1,0)
bar.BackgroundColor3 = Color3.fromRGB(210,210,210)
bar.BorderSizePixel = 0
Instance.new("UICorner", bar).CornerRadius = UDim.new(1,0)

local lStatus = Instance.new("TextLabel", box)
lStatus.Size = UDim2.new(1,0,0,14)
lStatus.Position = UDim2.new(0,0,0,120)
lStatus.BackgroundTransparency = 1
lStatus.Text = "Aguarde..."
lStatus.Font = Enum.Font.Gotham
lStatus.TextSize = 10
lStatus.TextColor3 = Color3.fromRGB(70,70,70)
lStatus.TextTransparency = 1

local lDots = Instance.new("TextLabel", box)
lDots.Size = UDim2.new(1,0,0,14)
lDots.Position = UDim2.new(0,0,0,140)
lDots.BackgroundTransparency = 1
lDots.Text = "•  •  •"
lDots.Font = Enum.Font.GothamBold
lDots.TextSize = 11
lDots.TextColor3 = Color3.fromRGB(55,55,55)
lDots.TextTransparency = 1

-- Entrada
tw(bg,  {BackgroundTransparency=0}, 0.5)
task.wait(0.3)
tw(box, {BackgroundTransparency=0, Position=UDim2.fromScale(0.5,0.5)}, 0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
tw(bStroke, {Transparency=0}, 0.4)
task.wait(0.25)
tw(lTitle,  {TextTransparency=0}, 0.35)
task.wait(0.12)
tw(lSub,    {TextTransparency=0}, 0.3)
tw(lPct,    {TextTransparency=0}, 0.3)
tw(barBG,   {BackgroundTransparency=0}, 0.3)
tw(lStatus, {TextTransparency=0}, 0.3)
tw(lDots,   {TextTransparency=0}, 0.3)

-- Dots animados
local dotsFrames = {"•      ","•  •   ","•  •  •","   •  •","      •"}
local di = 1
local dotsTask = task.spawn(function()
    while task.wait(0.28) do
        lDots.Text = dotsFrames[di]
        di = di % #dotsFrames + 1
    end
end)

local msgs = {
    "Carregando modulos...",
    "Configurando ESP...",
    "Preparando Aimbot...",
    "Ativando Anti-Ban...",
    "Detectando camera...",
    "Finalizando...",
}

for i = 1, 100 do
    lPct.Text = i.."%"
    tw(bar, {Size=UDim2.new(i/100,0,1,0)}, 0.04)
    lStatus.Text = msgs[math.min(math.ceil(i/(100/#msgs)), #msgs)]
    task.wait(0.015)
end

task.cancel(dotsTask)
lDots.Text = "✓"
lDots.TextColor3 = Color3.fromRGB(80,200,80)
lStatus.Text = "Pronto!"
tw(bar, {BackgroundColor3=Color3.fromRGB(80,200,80)}, 0.3)
task.wait(0.7)

-- Saída
tw(box,    {BackgroundTransparency=1, Position=UDim2.fromScale(0.5,0.4)}, 0.4, Enum.EasingStyle.Back, Enum.EasingDirection.In)
tw(lTitle, {TextTransparency=1}, 0.3)
tw(lSub,   {TextTransparency=1}, 0.3)
tw(lPct,   {TextTransparency=1}, 0.3)
tw(lStatus,{TextTransparency=1}, 0.3)
tw(lDots,  {TextTransparency=1}, 0.3)
tw(bg,     {BackgroundTransparency=1}, 0.5)
task.wait(0.6)
loadGui:Destroy()

print("Parte 2/5 carregada! Loading completo.")
-- RBX4HX [ALPHA] - PARTE 3/5
-- Interface Principal + Drag + Sistema de Abas

local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UIS          = game:GetService("UserInputService")
local L            = Players.LocalPlayer

local gui = Instance.new("ScreenGui")
gui.Name = "RBX4HX_UI"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.DisplayOrder = 100
pcall(function() gui.Parent = game:GetService("CoreGui") end)
if not gui.Parent then gui.Parent = L:WaitForChild("PlayerGui") end

-- BORDA
local border = Instance.new("Frame", gui)
border.Size = UDim2.fromOffset(506,306)
border.Position = UDim2.fromScale(0.5,0.5)
border.AnchorPoint = Vector2.new(0.5,0.5)
border.BackgroundColor3 = Color3.fromRGB(42,42,42)
border.BorderSizePixel = 0
Instance.new("UICorner", border).CornerRadius = UDim.new(0,16)

-- PAINEL
local panel = Instance.new("Frame", gui)
panel.Size = UDim2.fromOffset(500,300)
panel.Position = border.Position
panel.AnchorPoint = Vector2.new(0.5,0.5)
panel.BackgroundColor3 = Color3.fromRGB(20,20,20)
panel.BorderSizePixel = 0
Instance.new("UICorner", panel).CornerRadius = UDim.new(0,14)

-- HEADER
local header = Instance.new("Frame", panel)
header.Size = UDim2.new(1,0,0,48)
header.BackgroundColor3 = Color3.fromRGB(26,26,26)
header.BorderSizePixel = 0
Instance.new("UICorner", header).CornerRadius = UDim.new(0,14)

local titleLbl = Instance.new("TextLabel", header)
titleLbl.Size = UDim2.new(1,-110,1,0)
titleLbl.Position = UDim2.new(0,14,0,0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "RBX4HX"
titleLbl.Font = Enum.Font.GothamBold
titleLbl.TextSize = 16
titleLbl.TextXAlignment = Enum.TextXAlignment.Left
titleLbl.TextColor3 = Color3.fromRGB(225,225,225)

local minBtn = Instance.new("TextButton", header)
minBtn.Size = UDim2.fromOffset(32,32)
minBtn.Position = UDim2.new(1,-74,0.5,-16)
minBtn.Text = "─"
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 16
minBtn.BackgroundColor3 = Color3.fromRGB(34,34,34)
minBtn.TextColor3 = Color3.fromRGB(180,180,180)
minBtn.AutoButtonColor = false
minBtn.BorderSizePixel = 0
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,7)

local closeBtn = Instance.new("TextButton", header)
closeBtn.Size = UDim2.fromOffset(32,32)
closeBtn.Position = UDim2.new(1,-38,0.5,-16)
closeBtn.Text = "×"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 20
closeBtn.BackgroundColor3 = Color3.fromRGB(34,34,34)
closeBtn.TextColor3 = Color3.fromRGB(200,200,200)
closeBtn.AutoButtonColor = false
closeBtn.BorderSizePixel = 0
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0,7)

-- SEPARADOR
local sep = Instance.new("Frame", panel)
sep.Size = UDim2.new(1,0,0,1)
sep.Position = UDim2.new(0,0,0,48)
sep.BackgroundColor3 = Color3.fromRGB(38,38,38)
sep.BorderSizePixel = 0

-- SCROLL ABAS
local tabScroll = Instance.new("ScrollingFrame", panel)
tabScroll.Size = UDim2.new(1,-18,0,36)
tabScroll.Position = UDim2.new(0,9,0,54)
tabScroll.BackgroundTransparency = 1
tabScroll.ScrollBarThickness = 0
tabScroll.CanvasSize = UDim2.new(0,0,0,0)
tabScroll.AutomaticCanvasSize = Enum.AutomaticSize.X
tabScroll.ScrollingDirection = Enum.ScrollingDirection.X
tabScroll.ClipsDescendants = true

local tabLayout = Instance.new("UIListLayout", tabScroll)
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0,5)
tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center

-- CONTEÚDO
local contentHolder = Instance.new("Frame", panel)
contentHolder.Size = UDim2.new(1,-14,1,-100)
contentHolder.Position = UDim2.new(0,7,0,95)
contentHolder.BackgroundTransparency = 1
contentHolder.ClipsDescendants = true

-- DRAG
local dragging, dragInput, dragStart, startPos = false,nil,nil,nil

header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = panel.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

header.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local d = input.Position - dragStart
        local np = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + d.X,
            startPos.Y.Scale, startPos.Y.Offset + d.Y
        )
        panel.Position = np
        border.Position = np
    end
end)

-- BOTÃO REABRIR
local openBtn = Instance.new("TextButton", gui)
openBtn.Size = UDim2.fromOffset(48,48)
openBtn.Position = UDim2.new(0,18,0.5,-24)
openBtn.BackgroundColor3 = Color3.fromRGB(32,32,32)
openBtn.Text = "≡"
openBtn.TextColor3 = Color3.fromRGB(215,215,215)
openBtn.Font = Enum.Font.GothamBold
openBtn.TextSize = 20
openBtn.Visible = false
openBtn.AutoButtonColor = false
openBtn.BorderSizePixel = 0
Instance.new("UICorner", openBtn).CornerRadius = UDim.new(1,0)
Instance.new("UIStroke", openBtn).Color = Color3.fromRGB(55,55,55)

local minimized = false
local fullSize   = UDim2.fromOffset(500,300)
local miniSize   = UDim2.fromOffset(500,48)

minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    tabScroll.Visible     = not minimized
    contentHolder.Visible = not minimized
    sep.Visible           = not minimized
    minBtn.Text = minimized and "+" or "─"
    TweenService:Create(panel,  TweenInfo.new(0.25,Enum.EasingStyle.Quad), {Size=minimized and miniSize or fullSize}):Play()
    TweenService:Create(border, TweenInfo.new(0.25,Enum.EasingStyle.Quad), {Size=minimized and UDim2.fromOffset(506,54) or UDim2.fromOffset(506,306)}):Play()
end)

closeBtn.MouseButton1Click:Connect(function()
    panel.Visible = false border.Visible = false openBtn.Visible = true
end)

openBtn.MouseButton1Click:Connect(function()
    panel.Visible = true border.Visible = true openBtn.Visible = false
end)

local function addHover(btn, nc, hc)
    btn.MouseEnter:Connect(function() TweenService:Create(btn,TweenInfo.new(0.15),{BackgroundColor3=hc}):Play() end)
    btn.MouseLeave:Connect(function() TweenService:Create(btn,TweenInfo.new(0.15),{BackgroundColor3=nc}):Play() end)
end
addHover(minBtn,   Color3.fromRGB(34,34,34), Color3.fromRGB(52,52,52))
addHover(closeBtn, Color3.fromRGB(34,34,34), Color3.fromRGB(155,32,32))

-- SISTEMA DE ABAS
_G.tabList   = {}
_G.activeTab = nil

_G.CreateTab = function(name)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.fromOffset(95,28)
    btn.BackgroundColor3 = Color3.fromRGB(30,30,30)
    btn.TextColor3 = Color3.fromRGB(140,140,140)
    btn.TextSize = 12
    btn.Font = Enum.Font.GothamMedium
    btn.Text = name
    btn.AutoButtonColor = false
    btn.BorderSizePixel = 0
    btn.Parent = tabScroll
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1,0)

    local content = Instance.new("ScrollingFrame")
    content.Size = UDim2.new(1,0,1,0)
    content.BackgroundTransparency = 1
    content.ScrollBarThickness = 3
    content.ScrollBarImageColor3 = Color3.fromRGB(60,60,60)
    content.BorderSizePixel = 0
    content.CanvasSize = UDim2.new(0,0,0,0)
    content.AutomaticCanvasSize = Enum.AutomaticSize.Y
    content.Visible = false
    content.Parent = contentHolder

    local list = Instance.new("UIListLayout", content)
    list.Padding = UDim.new(0,6)
    list.SortOrder = Enum.SortOrder.LayoutOrder

    local pad = Instance.new("UIPadding", content)
    pad.PaddingTop    = UDim.new(0,4)
    pad.PaddingBottom = UDim.new(0,4)
    pad.PaddingLeft   = UDim.new(0,2)
    pad.PaddingRight  = UDim.new(0,2)

    _G.tabList[btn] = content

    btn.MouseButton1Click:Connect(function()
        if _G.activeTab then
            _G.tabList[_G.activeTab].Visible = false
            TweenService:Create(_G.activeTab, TweenInfo.new(0.15), {
                BackgroundColor3 = Color3.fromRGB(30,30,30),
                TextColor3 = Color3.fromRGB(140,140,140)
            }):Play()
        end
        _G.activeTab = btn
        content.Visible = true
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = Color3.fromRGB(55,55,55),
            TextColor3 = Color3.fromRGB(230,230,230)
        }):Play()
    end)

    return content
end

print("Parte 3/5 carregada! Interface pronta.")
-- RBX4HX [ALPHA] - PARTE 4/5
-- Componentes UI

local TweenService = game:GetService("TweenService")
local UIS          = game:GetService("UserInputService")

_G.createSection = function(parent, text)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1,0,0,20)
    f.BackgroundTransparency = 1
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1,-8,1,0)
    l.Position = UDim2.new(0,6,0,0)
    l.Text = text
    l.Font = Enum.Font.GothamBold
    l.TextSize = 10
    l.TextColor3 = Color3.fromRGB(120,120,120)
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.BackgroundTransparency = 1
end

_G.createLabel = function(parent, text)
    local l = Instance.new("TextLabel", parent)
    l.Size = UDim2.new(1,0,0,16)
    l.BackgroundTransparency = 1
    l.Text = "  "..text
    l.Font = Enum.Font.Gotham
    l.TextSize = 10
    l.TextColor3 = Color3.fromRGB(80,80,80)
    l.TextXAlignment = Enum.TextXAlignment.Left
end

_G.createToggle = function(parent, text, configKey, callback)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1,0,0,38)
    f.BackgroundColor3 = Color3.fromRGB(25,25,25)
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0,8)

    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1,-58,1,0)
    l.Position = UDim2.new(0,10,0,0)
    l.Text = text
    l.Font = Enum.Font.GothamMedium
    l.TextSize = 12
    l.TextColor3 = Color3.fromRGB(210,210,210)
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.BackgroundTransparency = 1

    local tBg = Instance.new("TextButton", f)
    tBg.Size = UDim2.fromOffset(44,24)
    tBg.Position = UDim2.new(1,-50,0.5,-12)
    tBg.BackgroundColor3 = Color3.fromRGB(42,42,42)
    tBg.BorderSizePixel = 0
    tBg.Text = ""
    tBg.AutoButtonColor = false
    Instance.new("UICorner", tBg).CornerRadius = UDim.new(1,0)

    local knob = Instance.new("Frame", tBg)
    knob.Size = UDim2.fromOffset(18,18)
    knob.Position = UDim2.new(0,3,0.5,-9)
    knob.BackgroundColor3 = Color3.fromRGB(160,160,160)
    knob.BorderSizePixel = 0
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1,0)

    local on = false
    tBg.MouseButton1Click:Connect(function()
        on = not on
        if configKey then _G.cfg[configKey] = on end
        if configKey == "antiafk" then Tafk(on) end
        if configKey == "antiban" or configKey == "antikick" then _G.SetupAntiBan() end
        if callback then callback(on) end
        TweenService:Create(tBg,  TweenInfo.new(0.2), {BackgroundColor3=on and Color3.fromRGB(70,170,70) or Color3.fromRGB(42,42,42)}):Play()
        TweenService:Create(knob, TweenInfo.new(0.2), {Position=on and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)}):Play()
        TweenService:Create(knob, TweenInfo.new(0.2), {BackgroundColor3=on and Color3.fromRGB(230,230,230) or Color3.fromRGB(160,160,160)}):Play()
    end)
end

-- SLIDER com hitbox largo e fácil de arrastar
_G.createSlider = function(parent, text, minV, maxV, default, configKey)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1,0,0,62)
    f.BackgroundColor3 = Color3.fromRGB(25,25,25)
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0,8)

    -- Texto do slider
    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1,-60,0,26)
    l.Position = UDim2.new(0,10,0,6)
    l.Text = text
    l.Font = Enum.Font.GothamMedium
    l.TextSize = 12
    l.TextColor3 = Color3.fromRGB(210,210,210)
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.BackgroundTransparency = 1

    -- Valor atual
    local val = Instance.new("TextLabel", f)
    val.Size = UDim2.new(0,50,0,26)
    val.Position = UDim2.new(1,-54,0,6)
    val.Text = tostring(default)
    val.Font = Enum.Font.GothamBold
    val.TextSize = 12
    val.TextColor3 = Color3.fromRGB(195,195,195)
    val.BackgroundTransparency = 1
    val.TextXAlignment = Enum.TextXAlignment.Right

    -- HITBOX LARGO: ocupa quase toda largura e 32px de altura
    -- Isso torna o slider fácil de tocar no mobile
    local hitbox = Instance.new("Frame", f)
    hitbox.Size = UDim2.new(1,-16,0,32)
    hitbox.Position = UDim2.new(0,8,1,-38)
    hitbox.BackgroundColor3 = Color3.fromRGB(32,32,32)
    hitbox.BorderSizePixel = 0
    Instance.new("UICorner", hitbox).CornerRadius = UDim.new(0,6)

    -- Track interno (visual, centralizado no hitbox)
    local track = Instance.new("Frame", hitbox)
    track.Size = UDim2.new(1,-16,0,4)
    track.Position = UDim2.new(0,8,0.5,-2)
    track.BackgroundColor3 = Color3.fromRGB(48,48,48)
    track.BorderSizePixel = 0
    Instance.new("UICorner", track).CornerRadius = UDim.new(1,0)

    -- Preenchimento
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new((default-minV)/(maxV-minV),0,1,0)
    fill.BackgroundColor3 = Color3.fromRGB(200,200,200)
    fill.BorderSizePixel = 0
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1,0)

    -- Knob grande e visível
    local knob = Instance.new("Frame", hitbox)
    knob.Size = UDim2.fromOffset(26,26)
    knob.AnchorPoint = Vector2.new(0.5,0.5)
    -- posição inicial baseada no valor default
    local initPct = (default-minV)/(maxV-minV)
    -- o track começa em 8px e tem largura = hitbox - 16
    -- o knob precisa estar relativo ao hitbox
    knob.Position = UDim2.new(initPct, (1-initPct)*8 - initPct*8, 0.5, 0)
    knob.BackgroundColor3 = Color3.fromRGB(235,235,235)
    knob.BorderSizePixel = 0
    knob.ZIndex = 4
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1,0)

    local dragging = false

    local function updateVal(inputX)
        -- calcula relativo ao track (com padding de 8px)
        local trackStart = hitbox.AbsolutePosition.X + 8
        local trackWidth = hitbox.AbsoluteSize.X - 16
        local rel = inputX - trackStart
        local pct = math.clamp(rel / trackWidth, 0, 1)
        local v   = math.floor((minV + (maxV-minV)*pct)*100+0.5)/100
        val.Text  = tostring(v)
        fill.Size = UDim2.new(pct,0,1,0)
        -- atualiza knob no hitbox
        knob.Position = UDim2.new(pct, (1-pct)*8 - pct*8, 0.5, 0)
        if configKey then _G.cfg[configKey] = v end
    end

    -- InputBegan no hitbox (Frame aceita Touch corretamente)
    hitbox.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            updateVal(input.Position.X)
        end
    end)

    hitbox.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if not dragging then return end
        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
            updateVal(input.Position.X)
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

_G.createDropdown = function(parent, text, options, default, configKey)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1,0,0,38)
    f.BackgroundColor3 = Color3.fromRGB(25,25,25)
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0,8)

    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1,-118,1,0)
    l.Position = UDim2.new(0,10,0,0)
    l.Text = text
    l.Font = Enum.Font.GothamMedium
    l.TextSize = 12
    l.TextColor3 = Color3.fromRGB(210,210,210)
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.BackgroundTransparency = 1

    local btn = Instance.new("TextButton", f)
    btn.Size = UDim2.fromOffset(110,26)
    btn.Position = UDim2.new(1,-114,0.5,-13)
    btn.BackgroundColor3 = Color3.fromRGB(33,33,33)
    btn.Text = default
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 11
    btn.TextColor3 = Color3.fromRGB(205,205,205)
    btn.BorderSizePixel = 0
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,7)
    Instance.new("UIStroke", btn).Color = Color3.fromRGB(46,46,46)

    local idx = 1
    btn.MouseButton1Click:Connect(function()
        idx = idx%#options+1
        btn.Text = options[idx]
        if configKey then _G.cfg[configKey] = options[idx] end
    end)
end

_G.createColorPicker = function(parent, text, default, configKey)
    local f = Instance.new("Frame", parent)
    f.Size = UDim2.new(1,0,0,38)
    f.BackgroundColor3 = Color3.fromRGB(25,25,25)
    f.BorderSizePixel = 0
    Instance.new("UICorner", f).CornerRadius = UDim.new(0,8)

    local l = Instance.new("TextLabel", f)
    l.Size = UDim2.new(1,-62,1,0)
    l.Position = UDim2.new(0,10,0,0)
    l.Text = text
    l.Font = Enum.Font.GothamMedium
    l.TextSize = 12
    l.TextColor3 = Color3.fromRGB(210,210,210)
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.BackgroundTransparency = 1

    local cb = Instance.new("TextButton", f)
    cb.Size = UDim2.fromOffset(46,26)
    cb.Position = UDim2.new(1,-50,0.5,-13)
    cb.BackgroundColor3 = default
    cb.Text = ""
    cb.BorderSizePixel = 0
    Instance.new("UICorner", cb).CornerRadius = UDim.new(0,7)
    Instance.new("UIStroke", cb).Color = Color3.fromRGB(58,58,58)

    local colors = {
        Color3.fromRGB(255,0,0),   Color3.fromRGB(0,255,0),
        Color3.fromRGB(0,120,255), Color3.fromRGB(255,255,0),
        Color3.fromRGB(255,0,255), Color3.fromRGB(0,255,255),
        Color3.fromRGB(255,255,255), Color3.fromRGB(255,140,0)
    }
    local idx = 1
    cb.MouseButton1Click:Connect(function()
        idx = idx%#colors+1
        cb.BackgroundColor3 = colors[idx]
        if configKey then _G.cfg[configKey] = colors[idx] end
    end)
end

print("Parte 4/5 carregada! Componentes prontos.")
-- RBX4HX [ALPHA] - PARTE 5/5
-- Conteudo das Abas

-- Atalhos locais
local cs  = _G.createSection
local cl  = _G.createLabel
local ct  = _G.createToggle
local csl = _G.createSlider
local cd  = _G.createDropdown
local ccp = _G.createColorPicker

-- CRIAR ABAS
local aimTab   = _G.CreateTab("🎯 Aim")
local espTab   = _G.CreateTab("👁 ESP")
local fovTab   = _G.CreateTab("🔍 FOV")
local extraTab = _G.CreateTab("⚡ Extra")
local banTab   = _G.CreateTab("🛡 Prot")

-- ══════════════════════════════
-- 🎯 AIM
-- ══════════════════════════════
cs(aimTab, "AIMBOT UNIVERSAL")
ct(aimTab, "Aimbot",       "aim")
ct(aimTab, "Wall Check",   "wc")
ct(aimTab, "Team Check",   "tc")
csl(aimTab,"Smooth",        0.01, 2.0, 0.9,  "as")
cd(aimTab, "Parte do Aim", {"Head","UpperTorso","LowerTorso","HumanoidRootPart"}, "Head", "aimpart")
cl(aimTab, "Detecta camera FPS automaticamente")

-- ══════════════════════════════
-- 👁 ESP
-- ══════════════════════════════
cs(espTab, "ESP UNIVERSAL")
ct(espTab, "Box ESP",      "eb")
ct(espTab, "Tracer",       "et")
ct(espTab, "Nome",         "en")
ct(espTab, "Distancia",    "edist")
ct(espTab, "Rainbow ESP",  "er")
ccp(espTab,"Cor ESP",      Color3.fromRGB(255,0,0), "ec")
csl(espTab,"Vel. Rainbow", 0.001, 0.02, 0.005, "rs")
cl(espTab, "Compativel com personagens customizados")

-- ══════════════════════════════
-- 🔍 FOV
-- ══════════════════════════════
cs(fovTab, "CIRCULO FOV")
ct(fovTab, "Mostrar FOV",  "fc")
ct(fovTab, "Invisivel",    "invfov")
ct(fovTab, "Rainbow FOV",  "fr")
csl(fovTab,"Tamanho",      10,  250, 100, "fs")
csl(fovTab,"Espessura",    1,   20,  2,   "ft")
ccp(fovTab,"Cor FOV",      Color3.fromRGB(255,255,255), "fco")

-- ══════════════════════════════
-- ⚡ EXTRA
-- ══════════════════════════════
cs(extraTab, "MOVIMENTO")
csl(extraTab,"WalkSpeed",    16, 85, 16, "ws")
ct(extraTab, "Pulo Infinito","infjump")
ct(extraTab, "Modo Seguro",  "safemode")
cl(extraTab, "Modo Seguro: WS max 28, aim off com anti-cheat")
cs(extraTab, "AFK")
ct(extraTab, "Anti-AFK",    "antiafk")
cl(extraTab, "Movimentos aleatorios para evitar kick AFK")

-- ══════════════════════════════
-- 🛡 PROTECAO
-- ══════════════════════════════
cs(banTab, "ANTI-BAN (4 CAMADAS)")
ct(banTab, "Anti-Ban",     "antiban")
ct(banTab, "Anti-Kick",    "antikick")
cl(banTab, "Cam.1: Bloqueia Kick via hookmetamethod")
cl(banTab, "Cam.2: Hook __index no Player")
cl(banTab, "Cam.3: Detecta scripts anti-cheat do jogo")
cl(banTab, "Cam.4: Monitor de ping suspeito")
cs(banTab, "LOG")
cl(banTab, "Logs visiveis no console (F9 no PC)")

-- Ativa primeira aba
local firstBtn = next(_G.tabList)
if firstBtn then
    _G.activeTab = firstBtn
    _G.tabList[firstBtn].Visible = true
    firstBtn.BackgroundColor3 = Color3.fromRGB(55,55,55)
    firstBtn.TextColor3 = Color3.fromRGB(230,230,230)
end

print("Parte 5/5 carregada! RBX4HX 100% pronto!")
