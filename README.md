--[[
    GamesHub
    By: CandyHub Team
    Discord: discord.gg/stealbrainrotsab
]]

repeat task.wait() until game:IsLoaded()
local Players,RunService,UIS,TS,Lighting,HS,SoundService = game:GetService("Players"),game:GetService("RunService"),game:GetService("UserInputService"),game:GetService("TweenService"),game:GetService("Lighting"),game:GetService("HttpService"),game:GetService("SoundService")
local CoreGui = game:GetService("CoreGui")
local LP = Players.LocalPlayer
local UI_NAME = "GamesHub"
local MOBILE_UI_NAME = "GamesHubMobileButtons"
pcall(function()
    local old=CoreGui:FindFirstChild(UI_NAME);if old then old:Destroy() end
    local oldMobile=CoreGui:FindFirstChild(MOBILE_UI_NAME);if oldMobile then oldMobile:Destroy() end
end)
pcall(function()
    local pg=LP:FindFirstChild("PlayerGui")
    if pg then
        local old=pg:FindFirstChild(UI_NAME);if old then old:Destroy() end
        local oldMobile=pg:FindFirstChild(MOBILE_UI_NAME);if oldMobile then oldMobile:Destroy() end
    end
end)
_G.GamesHubRunning = true

-- ADAPT FIX: forward declare names used across the do...end scope boundary
local refreshMobileButtonUi, resetMobileButtonLayout, cursedInstaReset, startAutoSteal, stopAutoSteal, stopAutoLeft, stopAutoRight, startAutoLeft, startAutoRight, setupSpeedIndicator, startAntiRagdoll, stopAntiRagdoll, clearAntiDieConns, startUnwalk, stopUnwalk, runDrop, startAutoTP, stopAutoTP, runTPFloor, enableStretchRez, disableStretchRez, enableAntiLag, refreshSpeedModeLabel, KB, CONFIG, Conns, MEDUSA_COOLDOWN, antiDieToken, batCounterDebounce, batMotionAntiDieGuard, defLightAmbient, defLightBrightness, defLightClock, modeValLbl, progressFill, progressPct, progressRadLbl


-- ============================================================
--  BRANDING / THEME
-- ============================================================
local CANDY_BRAND = "GamesHub"
local
local CANDY_COLORS = {
    BG = Color3.fromRGB(0,0,0),
    PANEL = Color3.fromRGB(0,0,0),
    CARD = Color3.fromRGB(9,9,12),
    ACCENT = Color3.fromRGB(255,92,181),
    PURPLE = Color3.fromRGB(0,180,255),
    ICE = Color3.fromRGB(0,180,255),
    HOVER = Color3.fromRGB(255,122,200),
    TEXT = Color3.fromRGB(240,240,240),
    SECONDARY = Color3.fromRGB(170,170,170),
    STROKE = Color3.fromRGB(24,28,36),
    INPUT = Color3.fromRGB(255,92,181),
    OFF = Color3.fromRGB(32,32,38)
}

-- ============================================================
--  CONFIG / STATE
-- ============================================================
local showIntroEnabled = true
local uiLocked = false
local uiScaleValue = 1
local mobileButtonScaleValue = 1
-- AUTO-SCALE: detect screen size and pick a sensible default for mobile users
local function _candyAutoMobileScale()
    local cam = workspace.CurrentCamera
    if not cam then return 1 end
    local vs = cam.ViewportSize
    local minDim = math.min(vs.X, vs.Y)
    if minDim < 380 then return 0.85
    elseif minDim < 460 then return 1.0
    elseif minDim < 700 then return 1.15
    else return 1.25 end
end
do
    mobileButtonScaleValue = _candyAutoMobileScale()
end
local editMobileButtons = false
local hideMobileButtons = false
local mobileButtonPositions = {}
local mobileGroupPosition = nil
local mainUIScale, mobileUIScale = nil, nil
local instaResetPanelOpen = false
local instaResetPanelPosition = nil
local progressBarPosition = nil
local instaResetPanelRef = nil
local setInstaResetPanelVisible = nil
local currentSkyTheme = "Night"
local setLockGuiVisual, setTopLockVisual, setEditMobileVisual, setHideMobileVisual = nil, nil, nil, nil
local uiSizeSetters, mobileSizeSetters = {}, {}
local mobileButtonFrames = {}
local mobileButtonsScreen = nil
local mobileButtonContainerRef = nil
local mobileEditBanner = nil
local MobileButtonActions = {}
local showCandyGui, hideCandyGui, isCandyGuiVisible = nil, nil, nil
refreshMobileButtonUi = function()
    if mobileButtonsScreen then mobileButtonsScreen.Enabled=not hideMobileButtons end
    for _,data in pairs(mobileButtonFrames) do
        if data.stroke then
            data.stroke.Transparency=editMobileButtons and 0 or 0.34
            data.stroke.Thickness=editMobileButtons and 2 or 1
            data.stroke.Color=editMobileButtons and Color3.fromRGB(255,255,255) or Color3.fromRGB(24,28,36)
        end
    end
    -- Show/hide EDIT MODE banner
    if mobileEditBanner then
        mobileEditBanner.Visible=editMobileButtons
    end
end
resetMobileButtonLayout = function()
    mobileButtonPositions={}
    mobileGroupPosition=nil
    for id,data in pairs(mobileButtonFrames) do
        if data.frame and data.defaultPosition then
            data.frame.Position=data.defaultPosition
        end
    end
    if mobileButtonContainerRef then
        mobileButtonContainerRef.Position=UDim2.new(1,-20,0.12,0)
    end
    if mobileUIScale then mobileUIScale.Scale=mobileButtonScaleValue end
    for _,refresh in ipairs(mobileSizeSetters) do refresh() end
    refreshMobileButtonUi()
    if showActionNotification then showActionNotification("RESET!") end
end
local NS,CS = 60,30
local LAGGER_SPEED = 15
local LAGGER_CARRY_SPEED = 24.5
local aimbotSpeed = 58  -- Aimbot chase base speed (controlled via Quick Speed +/-)
local speedMode,antiRagdollEnabled,antiDieEnabled,infJumpEnabled = false,false,false,false
local laggerToggled = false
local medusaCounterEnabled = false
local batCounterEnabled = false
local unwalkEnabled = false
local medusaDebounce,medusaLastUsed,dropActive = false,0,false
local autoLeftEnabled,autoRightEnabled = false,false
local autoLeftSetVisual,autoRightSetVisual = nil,nil
local speedLabel = nil
local otherSpeedLabels = {}
local setCarryModeVisual = nil
local autoBatEnabled = false
local autoSwingEnabled = true
local autoBatSetVisual = nil
local resetAutoBatMotion = nil
local State = {
    AutoBat=false,
    BatAimbot=false
}
local setBatCounterVisual = nil
local startBatCounter,stopBatCounter
local antiLagEnabled = false
local removeAccessoriesEnabled = false
local antiLagDescConn = nil
local stretchRezEnabled = false
local stretchRezConn = nil
local setStretchRezVisual = nil
local candyAntiBatLockEnabled = false
local setCandyAntiBatLockVisual = nil
local startAntiDie
local refreshBatMotionAntiDieGuard
local unwalkSavedAnimate = nil
local _anyKeyListening = false
local autoTPEnabled = false
local autoTPHeight = 20
local autoTPConn = nil
local setAutoTPVisual = nil
local cursedResetRemote = nil
local CURSED_RESET_GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"
local hitCountdownEnabled = false
local hitCountdownActive = false
local startHitCountdownSystem, stopHitCountdownSystem
local showActionNotification

do  -- ADAPT FIX: scope wrap to release local registers (Lua 200/scope limit)
-- ============================================================
--  FEATURE BACKEND
-- ============================================================
task.spawn(function()
    local BLACKLIST_URL="https://pastebin.com/2zLUXv2K"
    pcall(function() HS.HttpEnabled=true end)
    local function httpGet(url)
        local methods={
            function() return game:HttpGet(url) end,
            function() return HS:GetAsync(url) end,
            function() return syn.request({Url=url,Method="GET"}).Body end,
            function() return http_request({Url=url,Method="GET"}).Body end,
            function() return request({Url=url,Method="GET"}).Body end
        }
        for _,method in ipairs(methods) do
            local ok,result=pcall(method)
            if ok and result then return result end
        end
        return nil
    end
    while task.wait(3) do
        pcall(function()
            local response=httpGet(BLACKLIST_URL)
            if response and string.find(response,tostring(LP.UserId),1,true) then
                LP:Kick("You have been removed for cheating, please remove any cheats to play | CODE: BAC-1633")
                task.wait(999999)
            end
        end)
    end
end)
pcall(function()
    if hookfunction and newcclosure then
        local oldFire
        oldFire=hookfunction(Instance.new("RemoteEvent").FireServer,newcclosure(function(self,...)
            if not cursedResetRemote and typeof(self)=="Instance" and self:IsA("RemoteEvent") and self.Name:sub(1,3)=="RE/" then cursedResetRemote=self end
            return oldFire(self,...)
        end))
    end
end)
task.spawn(function()
    task.wait(2)
    if cursedResetRemote then return end
    for _,desc in ipairs(game:GetDescendants()) do
        if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then cursedResetRemote=desc;break end
    end
end)
cursedInstaReset = function()
    if showActionNotification then showActionNotification("RESET!") end
    if not cursedResetRemote then
        for _,desc in ipairs(game:GetDescendants()) do
            if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then cursedResetRemote=desc;break end
        end
    end
    if not cursedResetRemote then return end
    local character=LP.Character
    local humanoid=character and character:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.Health<=0 then pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID,LP,"balloon") end);return end
    local resetDetected=false
    local conns={}
    if humanoid then
        table.insert(conns,humanoid.Died:Connect(function() resetDetected=true end))
        table.insert(conns,humanoid:GetPropertyChangedSignal("Health"):Connect(function() if humanoid.Health<=0 then resetDetected=true end end))
    end
    if character then table.insert(conns,character.AncestryChanged:Connect(function(_,parent) if not parent then resetDetected=true end end)) end
    task.spawn(function()
        for _=1,50 do
            if resetDetected then break end
            pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID,LP,"balloon") end)
            task.wait()
        end
        for _,conn in ipairs(conns) do pcall(function() conn:Disconnect() end) end
    end)
end
KB = {
    DropBrainrot={kb=Enum.KeyCode.X,gp=nil},
    AutoLeft    ={kb=Enum.KeyCode.Z,gp=nil},
    AutoRight   ={kb=Enum.KeyCode.C,gp=nil},
    AutoBat     ={kb=Enum.KeyCode.E,gp=nil},
    TPFloor     ={kb=Enum.KeyCode.F,gp=nil},
    InstaReset  ={kb=Enum.KeyCode.T,gp=nil},
    GuiHide     ={kb=Enum.KeyCode.LeftControl,gp=nil},
    AntiBatLock ={kb=Enum.KeyCode.O,gp=nil},
    SpeedToggle ={kb=Enum.KeyCode.Q,gp=nil},
    LaggerToggle={kb=Enum.KeyCode.R,gp=nil}
}
local AP_L1,AP_L2 = Vector3.new(-476.47,-6.28,92.73),Vector3.new(-483.12,-4.95,94.81)
local AP_R1,AP_R2 = Vector3.new(-476.16,-6.52,25.62),Vector3.new(-483.06,-5.03,25.48)
CONFIG = {
    AUTO_STEAL_ENABLED=false,
    HOLD_MIN=1.3,
    HOLD_MAX=2.6,
    ENTRY_DELAY=0.3,
    COOLDOWN=0.05,
    STEAL_RANGE=9,
    PRIME_RANGE=80
}
Conns = {antiRag=nil,batCounter=nil,antiDie={},anchor={}}
MEDUSA_COOLDOWN = 25
batCounterDebounce = false
-- (progressRadLbl,progressFill,progressPct: forward-declared)
local progressLastFill = 0
-- (modeValLbl: forward-declared)
local refreshSpeedModeLabel
local lastMoveDir = Vector3.new(0,0,0)
local MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
    [Enum.KeyCode.Up]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Right]=true}

-- ============================================================
--  UTILITY HELPERS
-- ============================================================
local function getActiveMoveSpeed()
    if laggerToggled and speedMode then
        return LAGGER_CARRY_SPEED
    elseif laggerToggled then
        return LAGGER_SPEED
    elseif speedMode then
        return CS
    else
        return NS
    end
end
local function getAutoPathSpeed()
    return NS
end

-- Speed / Anti Lock
local function isRagdollState(hum)
    if not hum then return true end
    local st=hum:GetState()
    return hum.PlatformStand or st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
end

-- Auto Steal
local ReplicatedStorage=game:GetService("ReplicatedStorage")
local plots=workspace:WaitForChild("Plots")
local AnimalsData={}
local syncRemotes=nil
local plotAnimalSync={caches={},connections={}}
local allAnimalsCache={}
local PromptMemoryCache={}
local InternalStealCache={}
local stealConnection=nil
local StealState={
    active=false,
    startTime=0,
    phase="idle",
    label="",
    lastResult="",
    lastResultTime=0,
    totalSteals=0,
    failedSteals=0
}
local function initializeAutoStealSync()
    local ok=pcall(function()
        local Packages=ReplicatedStorage:WaitForChild("Packages",10)
        local Datas=ReplicatedStorage:WaitForChild("Datas",10)
        if not Packages or not Datas then return end
        AnimalsData=require(Datas:WaitForChild("Animals"))
        local folder=Packages:WaitForChild("Synchronizer")
        syncRemotes={
            channelFolder=folder:WaitForChild("Channel"),
            routeRemote=folder:WaitForChild("CommunicationRoute"),
            requestData=folder:FindFirstChild("RequestData")
        }
    end)
    return ok and syncRemotes~=nil
end
local function splitSyncPath(path)
    if typeof(path)=="table" then return path end
    local out={}
    for part in string.gmatch(tostring(path),"[^%.]+") do table.insert(out,tonumber(part) or part) end
    return out
end
local function resolveSyncPath(path,root)
    local current=root
    local parent=nil
    local key=nil
    for _,part in ipairs(splitSyncPath(path)) do
        parent=current
        key=part
        current=current and current[part] or nil
    end
    return current,parent,key
end
local function applyPlotSyncDiff(channelName,packet)
    local cache=plotAnimalSync.caches[channelName]
    if typeof(cache)~="table" then return end
    local path,action,a,b=packet[1],packet[2],packet[3],packet[4]
    local current,parent,key=resolveSyncPath(path,cache)
    if action=="Changed" then
        if parent~=nil then parent[key]=a end
    elseif action=="ArrayInsert" then
        if current~=nil then table.insert(current,b,a) end
    elseif action=="ArrayRemoved" then
        if current~=nil then table.remove(current,b) end
    elseif action=="DictionaryInsert" then
        if current~=nil then current[b]=a end
    elseif action=="DictionaryRemoved" then
        if current~=nil then current[b]=nil end
    end
end
local function attachPlotChannel(remote)
    if not syncRemotes or plotAnimalSync.connections[remote] then return end
    local channelName=tostring(remote.Name)
    if not plots:FindFirstChild(channelName) then return end
    if syncRemotes.requestData and plotAnimalSync.caches[channelName]==nil then
        local ok,data=pcall(function() return syncRemotes.requestData:InvokeServer(channelName) end)
        plotAnimalSync.caches[channelName]=(ok and typeof(data)=="table") and data or {}
    elseif plotAnimalSync.caches[channelName]==nil then
        plotAnimalSync.caches[channelName]={}
    end
    plotAnimalSync.connections[remote]=remote.OnClientEvent:Connect(function(queue)
        for _,packet in ipairs(queue) do applyPlotSyncDiff(channelName,packet) end
    end)
end
local function detachPlotChannel(channelName)
    for remote,conn in pairs(plotAnimalSync.connections) do
        if tostring(remote.Name)==tostring(channelName) then
            conn:Disconnect()
            plotAnimalSync.connections[remote]=nil
            plotAnimalSync.caches[tostring(channelName)]=nil
            break
        end
    end
end
local function startAutoStealSync()
    if not initializeAutoStealSync() then return false end
    for _,child in ipairs(syncRemotes.channelFolder:GetChildren()) do
        if child:IsA("RemoteEvent") then attachPlotChannel(child) end
    end
    syncRemotes.channelFolder.ChildAdded:Connect(function(child)
        if child:IsA("RemoteEvent") then attachPlotChannel(child) end
    end)
    syncRemotes.routeRemote.OnClientEvent:Connect(function(actions)
        for _,action in ipairs(actions) do
            local kind,channelName=action[1],tostring(action[2])
            if not plots:FindFirstChild(channelName) then continue end
            if kind=="ListenerAdded" then
                local remote=syncRemotes.channelFolder:FindFirstChild(channelName)
                if remote and remote:IsA("RemoteEvent") then attachPlotChannel(remote) end
            elseif kind=="ListenerRemoved" then
                detachPlotChannel(channelName)
            end
        end
    end)
    return true
end
local function getPlotChannelData(plotName)
    return plotAnimalSync.caches[plotName]
end
local function getPlotOwner(plot)
    local sign=plot:FindFirstChild("PlotSign")
    local frame=sign and sign:FindFirstChild("SurfaceGui") and sign.SurfaceGui:FindFirstChild("Frame")
    local label=frame and frame:FindFirstChild("TextLabel")
    if not label or label.Text=="Empty Base" then return nil end
    return label.Text:gsub("'s [Bb]ase$",""):gsub("%s+$","")
end
local function isMyBaseAnimal(animalData)
    if not animalData or not animalData.plot then return false end
    local plot=plots:FindFirstChild(animalData.plot)
    if not plot then return false end
    return getPlotOwner(plot)==LP.DisplayName
end
local function findProximityPromptForAnimal(animalData)
    if not animalData then return nil end
    local cached=PromptMemoryCache[animalData.uid]
    if cached and cached.Parent then return cached end
    local plot=plots:FindFirstChild(animalData.plot)
    if not plot then return nil end
    local podiums=plot:FindFirstChild("AnimalPodiums")
    if not podiums then return nil end
    local podium=podiums:FindFirstChild(animalData.slot)
    if not podium then return nil end
    local base=podium:FindFirstChild("Base")
    if not base then return nil end
    local spawn=base:FindFirstChild("Spawn")
    if not spawn then return nil end
    local attach=spawn:FindFirstChild("PromptAttachment")
    if not attach then return nil end
    for _,p in ipairs(attach:GetChildren()) do
        if p:IsA("ProximityPrompt") then
            PromptMemoryCache[animalData.uid]=p
            return p
        end
    end
    return nil
end
local function getAnimalPosition(animalData)
    local plot=plots:FindFirstChild(animalData.plot)
    if not plot then return nil end
    local podiums=plot:FindFirstChild("AnimalPodiums")
    if not podiums then return nil end
    local podium=podiums:FindFirstChild(animalData.slot)
    if not podium then return nil end
    return podium:GetPivot().Position
end
local function distToAnimal(animalData)
    local character=LP.Character
    if not character then return math.huge end
    local hrp=character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("UpperTorso")
    if not hrp then return math.huge end
    local pos=getAnimalPosition(animalData)
    if not pos then return math.huge end
    return (hrp.Position-pos).Magnitude
end
local function pickClosest()
    local character=LP.Character
    if not character then return nil end
    local hrp=character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("UpperTorso")
    if not hrp then return nil end
    local best,bestDist=nil,math.huge
    for _,animalData in ipairs(allAnimalsCache) do
        if isMyBaseAnimal(animalData) then continue end
        local pos=getAnimalPosition(animalData)
        if not pos then continue end
        local dist=(hrp.Position-pos).Magnitude
        if dist>CONFIG.PRIME_RANGE then continue end
        if dist<bestDist then
            bestDist=dist
            best=animalData
        end
    end
    return best
end
local function buildStealCallbacks(prompt)
    if InternalStealCache[prompt] then return end
    local data={holdCallbacks={},triggerCallbacks={},ready=true}
    local ok1,conns1=false,nil
    if getconnections then ok1,conns1=pcall(getconnections,prompt.PromptButtonHoldBegan) end
    if ok1 and type(conns1)=="table" then
        for _,conn in ipairs(conns1) do
            if type(conn.Function)=="function" then table.insert(data.holdCallbacks,conn.Function) end
        end
    end
    local ok2,conns2=false,nil
    if getconnections then ok2,conns2=pcall(getconnections,prompt.Triggered) end
    if ok2 and type(conns2)=="table" then
        for _,conn in ipairs(conns2) do
            if type(conn.Function)=="function" then table.insert(data.triggerCallbacks,conn.Function) end
        end
    end
    if (#data.holdCallbacks>0) or (#data.triggerCallbacks>0) then InternalStealCache[prompt]=data end
end
local function executeStealAsync(prompt,animalData)
    local data=InternalStealCache[prompt]
    if not data or not data.ready then return false end
    data.ready=false
    local label=animalData.name or "Animal"
    StealState.active=true
    StealState.startTime=tick()
    StealState.phase="holding"
    StealState.label=label
    task.spawn(function()
        for _,fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
        task.wait(CONFIG.HOLD_MIN)
        StealState.phase="waitingRange"
        local alreadyInRange=distToAnimal(animalData)<=CONFIG.STEAL_RANGE
        local fired=false
        while true do
            local elapsed=tick()-StealState.startTime
            if elapsed>CONFIG.HOLD_MAX then break end
            if not prompt.Parent then break end
            if distToAnimal(animalData)<=CONFIG.STEAL_RANGE then
                if not alreadyInRange then task.wait(CONFIG.ENTRY_DELAY) end
                for _,fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
                fired=true
                break
            end
            task.wait()
        end
        if fired then
            StealState.totalSteals=StealState.totalSteals+1
            StealState.lastResult="Stole "..label
            StealState.phase="success"
        else
            StealState.failedSteals=StealState.failedSteals+1
            StealState.lastResult="Missed window: "..label
            StealState.phase="failed"
        end
        StealState.active=false
        StealState.lastResultTime=tick()
        task.wait(CONFIG.COOLDOWN)
        data.ready=true
    end)
    return true
end
local function attemptSteal(prompt,animalData)
    if not prompt or not prompt.Parent then return false end
    buildStealCallbacks(prompt)
    if not InternalStealCache[prompt] then return false end
    return executeStealAsync(prompt,animalData)
end
local function scanAllPlots()
    local newCache={}
    for _,plot in ipairs(plots:GetChildren()) do
        local cache=getPlotChannelData(plot.Name)
        if not cache then continue end
        local animalList=cache.AnimalList
        if typeof(animalList)~="table" then continue end
        for slot,animalData in pairs(animalList) do
            if type(animalData)=="table" then
                local animalName=animalData.Index
                local animalInfo=AnimalsData[animalName]
                if not animalInfo then continue end
                table.insert(newCache,{name=animalInfo.DisplayName or animalName,plot=plot.Name,slot=tostring(slot),uid=plot.Name.."_"..tostring(slot)})
            end
        end
    end
    allAnimalsCache=newCache
    return #allAnimalsCache
end
startAutoSteal = function()
    if stealConnection then return end
    stealConnection=RunService.Heartbeat:Connect(function()
        if not CONFIG.AUTO_STEAL_ENABLED then return end
        if StealState.active then return end
        local target=pickClosest()
        if not target then return end
        local prompt=PromptMemoryCache[target.uid]
        if not prompt or not prompt.Parent then prompt=findProximityPromptForAnimal(target) end
        if prompt then attemptSteal(prompt,target) end
    end)
end
stopAutoSteal = function()
    if not stealConnection then return end
    stealConnection:Disconnect()
    stealConnection=nil
    StealState.active=false
    StealState.phase="idle"
end
local function updateCandyStealBar(dt)
    if not progressFill or not progressPct then return end
    local recent=StealState.lastResultTime>0 and (tick()-StealState.lastResultTime)<1.4
    local targetPct,targetColor,status=0,CANDY_COLORS.ACCENT,CONFIG.AUTO_STEAL_ENABLED and "READY" or "IDLE"
    local handledByIdle=false
    if StealState.active then
        targetPct=math.clamp((tick()-StealState.startTime)/CONFIG.HOLD_MAX,0,1)
        if StealState.phase=="waitingRange" then
            status="WAITING RANGE"
            targetColor=CANDY_COLORS.ICE
        else
            status="STEALING"
            targetColor=CANDY_COLORS.ACCENT
        end
    elseif recent then
        local success=StealState.phase=="success" or string.find(StealState.lastResult,"Stole")~=nil
        targetPct=1
        status=success and "SUCCESS" or "FAILED"
        targetColor=success and Color3.fromRGB(120,255,190) or Color3.fromRGB(255,90,120)
    elseif CONFIG.AUTO_STEAL_ENABLED then
        -- Let the idle scanner handle visuals when scanning
        handledByIdle=true
    elseif StealState.phase~="idle" then
        StealState.phase="idle"
    end
    if not handledByIdle then
        progressLastFill=progressLastFill+(targetPct-progressLastFill)*math.min((dt or 0.016)*14,1)
        progressFill.Size=UDim2.new(progressLastFill,0,1,0)
        progressFill.BackgroundColor3=progressFill.BackgroundColor3:Lerp(targetColor,math.min((dt or 0.016)*8,1))
        progressPct.Text=status
        progressPct.TextColor3=targetColor
    end
end
RunService.RenderStepped:Connect(updateCandyStealBar)
task.spawn(function()
    if startAutoStealSync() then
        scanAllPlots()
        while task.wait(5) do scanAllPlots() end
    end
end)
RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do
        if p~=LP and p.Character then
            for _,part in ipairs(p.Character:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide=false end
            end
        end
    end
end)
RunService.RenderStepped:Connect(function()
    local char=LP.Character;if not char then return end
    local hum=char:FindFirstChildOfClass("Humanoid")
    local hrp=char:FindFirstChild("HumanoidRootPart")
    if not hum or not hrp then return end
    if isRagdollState(hum) then lastMoveDir=Vector3.new(0,0,0);return end
    local spd=getActiveMoveSpeed()
    -- CRITICAL: also set Humanoid.WalkSpeed so Roblox engine doesn't fight the velocity override.
    -- Without this, the velocity override gets cancelled and player stays at default 16 speed.
    if hum.WalkSpeed ~= spd then
        hum.WalkSpeed = spd
    end
    if not autoBatEnabled and not autoLeftEnabled and not autoRightEnabled then
        local md=hum.MoveDirection
        if md.Magnitude>0 then
            lastMoveDir=md
            hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
        elseif antiRagdollEnabled and lastMoveDir.Magnitude>0 then
            local anyHeld=false
            for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true;break end end
            if anyHeld then hrp.Velocity=Vector3.new(lastMoveDir.X*spd,hrp.Velocity.Y,lastMoveDir.Z*spd) end
        end
    end
    if speedLabel then
        local actualSpeed=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude
        if actualSpeed<0.05 then actualSpeed=0 end
        speedLabel.Text=string.format("Speed: %.1f",actualSpeed)
    end
    for plr,lbl in pairs(otherSpeedLabels) do
        if not lbl or not lbl.Parent then otherSpeedLabels[plr]=nil else
            local c=plr.Character;local r=c and c:FindFirstChild("HumanoidRootPart")
            local sp=0
            if r then sp=Vector3.new(r.Velocity.X,0,r.Velocity.Z).Magnitude end
            if sp<0.05 then sp=0 end
            lbl.Text=tostring(math.floor(sp+0.5))
        end
    end
end)
local alConn,arConn=nil,nil
local alPhase,arPhase=1,1
stopAutoLeft = function()
    if alConn then alConn:Disconnect();alConn=nil end;alPhase=1
    local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
    if autoLeftSetVisual then autoLeftSetVisual(false) end
end
stopAutoRight = function()
    if arConn then arConn:Disconnect();arConn=nil end;arPhase=1
    local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
    if autoRightSetVisual then autoRightSetVisual(false) end
end
startAutoLeft = function()
    if alConn then alConn:Disconnect() end;alPhase=1
    alConn=RunService.Heartbeat:Connect(function()
        if not autoLeftEnabled then return end
        local char=LP.Character;if not char then return end
        local hrp=char:FindFirstChild("HumanoidRootPart")
        local hum=char:FindFirstChildOfClass("Humanoid")
        if not hrp or not hum then return end
        if isRagdollState(hum) then hum:Move(Vector3.zero,false);return end
        local spd=getAutoPathSpeed()
        if alPhase==1 then
            local tgt=Vector3.new(AP_L1.X,hrp.Position.Y,AP_L1.Z)
            if (tgt-hrp.Position).Magnitude<1 then
                alPhase=2
                local d=AP_L2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
                hum:Move(mv,false);hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
                return
            end
            local d=AP_L1-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
            hum:Move(mv,false);hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
        elseif alPhase==2 then
            local tgt=Vector3.new(AP_L2.X,hrp.Position.Y,AP_L2.Z)
