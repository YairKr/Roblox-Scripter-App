# Roblox-Scripter-App

-- hello this is my application for a ranked scripter role 
-- in the provided example i will show you diffrent approuch to scripting 
-- error handling is the first 
-- abstract objects with oop is the secoend 
-- effiecnt work and maximizing memory in  way 

-- almost all is done with a networking library to maximize effiecenty 

-- if you need more work / video of proof i will provide if asked 

-- have a good day if you have any questions please ask me 

Krovense on discord 
have a good day 

Application

-- for remotes i use suphi packets and adjusted the remotes buffers for maximum efficeny 
-- so the remote already know its gonna recive a string : string and a table of addons {} : its too long 

--Main module script give you the corrcet module based on device 
-- Client / server 
```
local RunService = game:GetService("RunService")

local VFXHandler = nil	

if RunService:IsClient() then
	VFXHandler = require(script.Client)

	if script:FindFirstChild("Server") then
		script:FindFirstChild("Server"):Destroy()
	end

	VFXHandler.Setup()
end

if RunService:IsServer() then
	VFXHandler = require(script.Server)
end	


return VFXHandler
```
----------------------------------------------------------------
--ServerSide just to run the commands its insdie the module 
```
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local SharedModules = ReplicatedStorage.SharedModules
local Components = SharedModules.Components

local Packets = require(ReplicatedStorage.Packets)

local ServerEffectrsHandler = {}

function ServerEffectrsHandler.Run(EffectName : string,Arguments : {})
	Packets.EffectsHandler:Fire(EffectName,Arguments)
end

function ServerEffectrsHandler.RunOnly(Player : Player,EffectName : string,Arguments : {})
	Packets.EffectsHandler:FireClient(Player,EffectName,Arguments)
end

return ServerEffectrsHandler
```
---------------------------------------------------------------
--CLient side 
```
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")

local SharedModules = ReplicatedStorage.SharedModules
local Components = SharedModules.Components

local EffectsFolder = Components.Effects

local Packets = require(ReplicatedStorage.Packets)

local Player : Player = Players.LocalPlayer

local Effects = {}

local ClientEffectsHandler = {}


-- Sets up all the modules of vfx / effects

function ClientEffectsHandler.Setup()

	for _ ,Module in EffectsFolder:GetDescendants() do
		if Module:IsA("ModuleScript") then
			local Req = nil
			xpcall(function()
				Req = require(Module)
			end,function(eror)
				task.spawn(function() -- error handling
					error("clientfx: error loading '" .. Module:GetFullName() .. "': " .. eror)
				end)
			end)

			if Req then
				for FuncName, Function in Req do
					Effects[FuncName] = Function
				end
			end

		end
	end

end

function ClientEffectsHandler.Run(EffectName : string,Arguments : {})
	if not Arguments then
		Arguments = {}
	end
	local Function = Effects[EffectName]
	if Function then

		-- error handling + debug tracback for easier fixing 

		xpcall(function() 
			Function(Arguments)
		end, function(Error)
			local TraceBack = debug.traceback()
			task.spawn(function()
				warn("ClientFX: effect '" .. EffectName .. "' threw exception: " .. Error)
				print(TraceBack)
			end)
		end)
	end
end

Packets.EffectsHandler.OnClientEvent:Connect(ClientEffectsHandler.Run)

return ClientEffectsHandler
```
-------------------------------------------------------------------------------------------------------
--Example of effects really simple just simple movements 
```
local Debris = game:GetService("Debris")
local RunService = game:GetService("RunService")

local Movements = {}


function Movements.SprintMomentum(Arguments)
	local character = Arguments.Character
	local HumanoidRootPart : BasePart = character.HumanoidRootPart

	local StartClock = os.clock()
	local TimerConnection : RBXScriptConnection

	local LinearVelocity = Instance.new("LinearVelocity")
	local Attachment = Instance.new("Attachment")

	LinearVelocity.ForceLimitsEnabled = true
	LinearVelocity.ForceLimitMode = Enum.ForceLimitMode.PerAxis
	LinearVelocity.MaxAxesForce = Vector3.new(math.huge,0,math.huge)
	LinearVelocity.VectorVelocity = HumanoidRootPart.CFrame.LookVector * 30 

	LinearVelocity.Attachment0 = Attachment
	LinearVelocity.Parent = HumanoidRootPart
	Attachment.Parent = HumanoidRootPart

	Debris:AddItem(LinearVelocity,0.3)
	Debris:AddItem(Attachment,0.3)

	TimerConnection = RunService.RenderStepped:Connect(function()

		if StartClock - os.clock() >= 0.3 then
			TimerConnection:Disconnect()
			return
		end	
		LinearVelocity.VectorVelocity = HumanoidRootPart.CFrame.LookVector * 30
	end)

end

function Movements.AerialMomentum(Arguments)
	local character = Arguments.Character
	local HumanoidRootPart : BasePart = character.HumanoidRootPart

	local StartClock = os.clock()
	local TimerConnection : RBXScriptConnection

	local LinearVelocity = Instance.new("LinearVelocity")
	local Attachment = Instance.new("Attachment")

	LinearVelocity.ForceLimitsEnabled = false
	LinearVelocity.MaxForce = math.huge
	LinearVelocity.VectorVelocity = HumanoidRootPart.CFrame.LookVector * 45 

	LinearVelocity.Attachment0 = Attachment
	LinearVelocity.Parent = HumanoidRootPart
	Attachment.Parent = HumanoidRootPart

	Debris:AddItem(LinearVelocity,0.3)
	Debris:AddItem(Attachment,0.3)

	TimerConnection = RunService.RenderStepped:Connect(function()

		if StartClock - os.clock() >= 0.3 then
			TimerConnection:Disconnect()
			return
		end	
		LinearVelocity.VectorVelocity = HumanoidRootPart.CFrame.LookVector * 45 + Vector3.new(0,-15,0)
	end)
end

function Movements.FallingDragon(Arguments)
	local character = Arguments.Character
	local TargetPos = Arguments.TargetPos

	local HumanoidRootPart : BasePart = character.HumanoidRootPart

	local StartClock = os.clock()
	local TimerConnection : RBXScriptConnection

	local AlignPosition2 = Instance.new("AlignPosition")
	AlignPosition2.Mode = Enum.PositionAlignmentMode.OneAttachment
	AlignPosition2.Attachment0 = HumanoidRootPart.RootAttachment
	AlignPosition2.Position = TargetPos 
	AlignPosition2.Responsiveness = 10
	AlignPosition2.RigidityEnabled = true
	AlignPosition2.Parent = HumanoidRootPart

	TimerConnection = RunService.RenderStepped:Connect(function()

		if (HumanoidRootPart.Position - TargetPos).Magnitude < 3.5 then

			AlignPosition2:Destroy()
			game.ReplicatedStorage.Remotes.FallingDragon:FireServer()
			TimerConnection:Disconnect()
			return
		end

	end)
end

return Movements
```


---------------------------------------------------------------------------------------------

This thing shows a diffrent approch to scripting than showed above

--Bonus to show i know how to handle abstract things instead of practical only 
-- i tried to recreate deepwoken effecthander (Stun , ragdoll etc...)
--stay with me its a bit complicated to explain  lol


-- lets start with the server side module
```
local Http = game:GetService("HttpService")

local EffectReplicator = {}


local function CreateSignal()
	local Signal = Instance.new("BindableEvent")
	Signal.Name = "Shadow"
	return Signal
end

function EffectReplicator.New(Self,LivingObject) -- when characrter joins / get created it creates a shell that will store the effects
	
	local Shell = {}
	Shell.Being = LivingObject
	Shell.Effects = {}
	Shell.Container = LivingObject
	Shell.EffectAdded = CreateSignal()
	Shell.EffectRemoved = CreateSignal()

	
	--using meta tables to confuse infinity yield users... jk jk

	setmetatable(Shell , EffectReplicator)
	EffectReplicator.__index = EffectReplicator

	return Shell	

end

 -- im calling from other scripts with : so its sends it self first argument
--  i just find it more readable than just using self normally
function EffectReplicator.CreateEffect(Self, EffectName, Arguments)

	if not Arguments then Arguments = {} end

	local NewEffect = {}
	NewEffect.Owner = EffectReplicator.Container
	NewEffect.Class = EffectName
	NewEffect.Value = Arguments.Value or true 
	NewEffect.Tags = Arguments.Tags or {}
	NewEffect.ID =  Http:GenerateGUID()
	NewEffect.Domain =  "Server"
	NewEffect.Disabled = Arguments.Disabled or false
	
	--creates a new effect with a unique id
	--and gets the domain


	-- meta tables to treat every effect as an object 
	-- makes it easier to edit it on the run in game 
	-- with just destory / edit the debris time
	local Shell = {
		Index = NewEffect	
	}

	function Shell.Destroy(Effect)
		Self.EffectRemoved:Fire(Effect)

		Self.Effects[Effect.ID] = nil

	end
	
	function Shell.Debris(Self,Time,Update)
		if not Time then Time = 0 end

		Self.DebrisTime = Time


		local T = Time
		task.delay(Time,function()
			if Self then
				if T == Self.DebrisTime then
					Self:Destroy()
				end
			end

		end)

	end

	Self.Effects[NewEffect.ID] = NewEffect

	setmetatable(NewEffect,{__index = Shell})
	
	-- now for i recreated collectionservice methods to make it easeir to use

	if Self.EffectAdded then
		Self.EffectAdded:Fire(NewEffect)
	end
	return NewEffect
end

--gets effects

function EffectReplicator.GetEffects(Self)
	return Self.Effects
end

-- gets minimal data for the effects

function EffectReplicator.GetStrippedEffects(Self)

	local ShellEffects = Self.Effects 
	local ToGive = {}

	for i , EffectData in ShellEffects do
		table.insert(ToGive,{
			Class = EffectData.Class,
			Disabled = EffectData.Disabled or nil,
			Tags = EffectData.Tags or nil,
			Domain = EffectData.Domain,
			ID = EffectData.ID,
			Value = EffectData.Value or nil,
			DebrisTime = EffectData.DebrisTime or nil,	
		})

	end
	return ToGive
end

-- find effect based on name (class)
function EffectReplicator.FindEffect(Self,Class)
	for i ,Effect in Self.Effects do

		if Effect.Class == Class then
			if not Effect.Disabled then
				return Effect
			end
		end

	end
	return nil
end

-- clears an entire class 
--so if i have multiple slow effects and i get a cc immune potion i can just clear all slows

function EffectReplicator.ClearClass(Self,Class,IgnoreDisabled)
	for i ,Effect in Self.Effects do
		if Effect.Class == Class then
			if IgnoreDisabled then
				if not Effect.Disabled then
					Effect:Destroy()
				end
			else
				Effect:Destroy()
			end
		end

	end
end

-- effects can store values like slow = -16
-- and tags , tags is like values but another way to store information primarly used for strings
function EffectReplicator.GetEffectsWithTag(Self,tag)
	local effectsArray = {}

	for _, effect in next, Self.Effects do
		if (effect.Tags[tag]) then
			table.insert(effectsArray, effect)
		end
	end

	return effectsArray
end




return EffectReplicator

```
------------------------------------------------------------------------
-- now server script 
```
local function PlayerAdded(Player)

	PlayersData[Player] = DataStore:CreateShell(Player,{Player.UserId})
	PlayersConnection[Player] = {}

	local Slot = PlayersData[Player].Slot

	table.insert(PlayersConnection[Player], Player.CharacterAdded:Connect(function(Character)

		-- this part is importnat the CreateReplicator
		shared.CreateReplicator(Character)
		shared.CreateAnimate(Character)

		while Character.Parent ~= Live do
			Character.Parent = Live
			task.wait(0.1)
		end

		CharacterEdit.SetAttributes(Player,Character,Slot)
		CharacterEdit.Customize(Player,Character) 

		PlayersData[Player].Character = Character

		Character:SetAttribute("Loaded",true)

	end))



	Player:SetAttribute("NotSpawned",true)
end
```
-------------------------------------------------------------------
--Create replicator 
```
function shared.CreateReplicator(LivingObject)
	local IsPlayer = Players:GetPlayerFromCharacter(LivingObject)

	local Shell = EffectReplicator:New(LivingObject)

	PlayerEffects[LivingObject] = {}
	PlayerEffects[LivingObject].Replicator = Shell

	if IsPlayer then


		local EffectAddedConnection 
		local EffectRemovedConnection 

		-- on effect added
		EffectAddedConnection = Shell.EffectAdded.Event:Connect(function(Effect)

			UpdateClientEffect:FireClient(IsPlayer ,{
				Type = "Update",
				Effects = Shell:GetStrippedEffects(),
			})
		end)
		-- on effect removed
		EffectRemovedConnection = Shell.EffectRemoved.Event:Connect(function(Effect)
			UpdateClientEffect:FireClient(IsPlayer  ,{
				Type = "Remove",
				EffectID = Effect.ID,
			})
		end)
		PlayerEffects[LivingObject].AddedConnection = EffectAddedConnection
		PlayerEffects[LivingObject].RemovedConnection = EffectRemovedConnection

		

		-- to replicate to the client 
		UpdateClientEffect:FireClient(IsPlayer  ,{
			Type = "SetContainer",
			Container = IsPlayer,
		})


	end



	return Shell
end

function shared.GetReplicator(LivingObject)
	return PlayerEffects[LivingObject] and PlayerEffects[LivingObject].Replicator
end
```
---------------------------------------------------------------------------------------
-- client main module
--basiclly a recreation of the server just to fit the client
```
local Http = game:GetService("HttpService")

local EffectReplicator = {

	Effects = {},
	Container = nil,

}


function EffectReplicator.CreateEffect(Self, EffectName, Arguments)

	if not Arguments then Arguments = {} end

	local NewEffect = {}
	NewEffect.Owner = EffectReplicator.Container
	NewEffect.Class = EffectName
	NewEffect.Value = Arguments.Value or true
	NewEffect.Tags = Arguments.Tags or {}
	NewEffect.ID = Arguments.ID or Http:GenerateGUID()
	NewEffect.Domain = Arguments.Domain or "Client"
	NewEffect.Disabled = Arguments.Disabled or false


	local Shell = {
		Index = NewEffect	
	}

	function Shell.Destroy()

		if Self.EffectRemoved and Self.EffectRemoved.Shadow then
			Self.EffectRemoved.Shadow:Fire(NewEffect)
		end



		Self.Effects[NewEffect.ID] = nil

	end

	function Shell.Debris(Self,Time)
		if not Time then Time = 0 end

		NewEffect.DebrisTime = Time


		delay(Time,function()
			if NewEffect then
				NewEffect:Destroy()
			end

		end)

	end



	setmetatable(NewEffect,{__index = Shell})

	Self.Effects[NewEffect.ID] = NewEffect

	--[[print(NewEffect.Class)
	print(EffectReplicator.Effects)
	print(Self.Effects)]]

	if Self.EffectAdded and Self.EffectAdded.Shadow then
		Self.EffectAdded.Shadow:Fire(NewEffect)
	end
	return NewEffect
end


function EffectReplicator.FindEffectByID(Self,EffectID)
	return Self.Effects[EffectID]
end

function EffectReplicator:GetEffect(Effect)
	return EffectReplicator.Effects[Effect.ID or Effect]
end

function EffectReplicator.GetStrippedEffects(Self)

	local ShellEffects = Self.Effects
	local ToGive = {}


	for i , EffectData in ShellEffects do
		table.insert(ToGive,{
			Class = EffectData.Class,
			Disabled = EffectData.Disabled or nil,
			Tags = EffectData.Tags or nil,
			Domain = EffectData.Domain,
			ID = EffectData.ID,
			Value = EffectData.Value or nil,
			DebrisTime = EffectData.DebrisTime or nil,	
		})

	end
	return ToGive
end

function EffectReplicator.FindEffect(Self,Class)
	for i ,Effect in Self.Effects do
		if Effect.Class == Class then
			if not Effect.Disabled then
				return Effect
			end
		end

	end
	return nil
end

function EffectReplicator.GetEffects(Self)
	return Self.Effects
end

function EffectReplicator.ClearClass(Self,Class,IgnoreDisabled)
	for i ,Effect in Self.Effects do
		if Effect.Class == Class then
			if IgnoreDisabled then
				if not Effect.Disabled then
					Effect:Destroy()
				end
			else
				Effect:Destroy()
			end
		end

	end
end

function EffectReplicator.GetEffectsWithTag(Self,tag)
	local effectsArray = {}

	for _, effect in next, Self.Effects do
		if (effect.Tags[tag]) then
			table.insert(effectsArray, effect)
		end
	end

	return effectsArray
end

function EffectReplicator.WaitForContainer(Self)
	while EffectReplicator.Container == nil do
		print(EffectReplicator.Container)
		task.wait()
	end
	return Self
end


local Connector = {

	Connect = function(Self,CallBack)

		if type(CallBack) ~= "function" then
			return
		end
		local Shadow = Self.Shadow
		if not Shadow then
			Shadow = Instance.new("BindableEvent")
			Shadow.Name = "ContainerShadowBindable"
			Self.Shadow = Shadow
		end
		return Shadow.Event:connect(function(Effect)
			local ToGive = EffectReplicator:GetEffect(Effect)
			if not ToGive then
				return
			end
			CallBack(ToGive)
		end)


	end,


}

-- really just a diffrent way instead of using bindables really make it easier while typing and 
-- im pretty sure its faster cause of the callback method havnt tested preformace tho



local EffectAdded = {}
local EffectRemoved = {}
setmetatable(EffectAdded, {
	__index = Connector
})
setmetatable(EffectRemoved, {
	__index = Connector
})
EffectReplicator.EffectAdded = EffectAdded
EffectReplicator.EffectRemoved = EffectRemoved

game.ReplicatedStorage.Requests.EffectReplication:WaitForChild("_update").OnClientEvent:connect(function(Arguments)
	if not Arguments.Type then return end

	local Type = Arguments.Type
	local Effects , EffectID , Container = Arguments.Effects or nil , Arguments.EffectID or nil , Arguments.Container or nil


	if Type == "Update" then
		for _ , Effect in Effects do

			local ClientEffect = EffectReplicator:GetEffect(Effect.ID)

			if ClientEffect then



				ClientEffect.Value = Effect.Value
				ClientEffect.DebrisTime = Effect.DebrisTime
				if Effect.Disabled ~= nil then
					ClientEffect.Disabled = Effect.Disabled
				end
				if Effect.Tags ~= nil then
					ClientEffect.Tags = Effect.Tags
				end



			else

				local NewEffect = EffectReplicator:CreateEffect(Effect.Class,
					{Value = Effect.Value ,
						Tags = Effect.Tags,
						Doamin = Effect.Domain,
						ID = Effect.ID
					})
				if Effect.DebrisTime then
					NewEffect:Debris(Effect.DebrisTime or 0,true)
				end



			end



		end

	elseif Type == "Remove" then
		local ToRemove = EffectReplicator:FindEffectByID(EffectID)

		if ToRemove then
			ToRemove:Destroy()
		end

	elseif Type == "SetContainer" then
		if Container then
			EffectReplicator.Container = Container
			print("Hello contianer set")
			print(EffectReplicator.Container)
		end
	end


end)




return EffectReplicator
```

-----------------------------------------------------------------------------------------
--i will create a very simple way to implment them with a local script 
-- the beauty of this way of handling stun is its almost exploiter prof i mean how can an exploiter know if you are attacking if he cant see your client
-- its one of its many bonuses but just food for thought

```

local GetDescendants = game.GetDescendants
local IsA = game.IsA
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local EffectsMod = require(ReplicatedStorage.EffectReplicator)

local Player = game.Players.LocalPlayer
local Character = Player.Character
local Humanoid = Character:WaitForChild("Humanoid")
local Head = Character:WaitForChild("Head")
local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")

local Rand = Random.new()
Humanoid.PlatformStand = true

local Focus = Instance.new("DepthOfFieldEffect",game.Lighting)

local CurrentCamera = workspace.CurrentCamera


local IsShiftLock = nil
local CameraToggle = false

local States = { Enum.HumanoidStateType.Dead, Enum.HumanoidStateType.FallingDown, Enum.HumanoidStateType.Ragdoll, Enum.HumanoidStateType.Seated, Enum.HumanoidStateType.Climbing, Enum.HumanoidStateType.Swimming, Enum.HumanoidStateType.Flying, Enum.HumanoidStateType.StrafingNoPhysics, Enum.HumanoidStateType.GettingUp }
local function setupHumanoid(Humanoid)
	for i, v in next, States do
		Humanoid:SetStateEnabled(v, false)
	end
end

local function EffectsFunction(Beat)

	local Speed = 16
	local Jump = 7.2
	local SpeedNerf = false
	local JumpNerf = false
	local AutoRotate = true
	local PlatformStand = false
	local FieldOfView = 70
	local CameraMaxZoomDistance = 20
	local LightingBlur = 0
	local DepthOfField = false


	local HpDifference = Humanoid.Health / Humanoid.MaxHealth
	local stopCameraUpdate = false

	local TotalEffects = EffectsMod:GetStrippedEffects()


	for EffectIndex , Effect in TotalEffects do
		if Effect.Disabled then continue end


		if Effect.Class == "Speed" or Effect.Class == "SpeedUp" and not SpeedNerf then

			Speed = Speed + Effect.Value

		end


		if Effect.Class == "CharacterCreation" then
			Speed = 0
			Jump = 0

			stopCameraUpdate = true
			CameraToggle = true
			AutoRotate = false
			SpeedNerf = true
			JumpNerf = true



		elseif Effect.Class == "Sprinting" and not SpeedNerf then
			local Boost = Effect.Value + 4 * (Humanoid.Health / Humanoid.MaxHealth)

			Speed = Speed + Boost



		elseif Effect.Class == "NoMovement" then


			Speed = 0
			Jump = 0



			SpeedNerf = true
			JumpNerf = true

		elseif Effect.Class == "Stun" then
			Jump = 0
			JumpNerf = true


		elseif Effect.Class == "Test" then

			--print("Test")


		elseif Effect.Class == "NoRotate" then
			AutoRotate = false

		elseif Effect.Class == "Blocking" then

			Speed = Speed - 5

		elseif Effect.Class == "Rest" then

			SpeedNerf = true
			JumpNerf = true

			Speed = 0
			Jump = 0

			AutoRotate = false
			DepthOfField = true
		end

	end

	Humanoid.WalkSpeed = math.max(0, Speed)

	Jump = math.max(0,Jump)

	Humanoid.JumpHeight = Jump
	if Jump == 0 then
		Humanoid:SetStateEnabled("Jumping", false)
		Humanoid.Jump = false
	else
		Humanoid:SetStateEnabled("Jumping", true)
	end
	Humanoid.AutoRotate = AutoRotate
	Humanoid.PlatformStand = PlatformStand


	if not stopCameraUpdate then
		if Humanoid.Health > 0  then
			CurrentCamera.CameraSubject = Humanoid

		end
		CurrentCamera.CameraType = CameraToggle and Enum.CameraType.Scriptable or Enum.CameraType.Custom
		CurrentCamera.FieldOfView = FieldOfView

	end

	Focus.Enabled = DepthOfField

	if game:GetService("UserInputService").MouseBehavior == Enum.MouseBehavior.LockCenter then
		if IsShiftLock then
			return
		end
	else
		if IsShiftLock then
			IsShiftLock:Destroy()
			IsShiftLock = nil
		end
		return
	end
	IsShiftLock = EffectsMod:CreateEffect("ShiftLock")
end


setupHumanoid(Humanoid)




game:service("RunService").RenderStepped:connect(function(Beat)
	local success, Eror = pcall(EffectsFunction, Beat)
	if not success then
		warn(Eror)
	end
end)
```
--------------------------------------------------
-- a way to exploit is to remove / disable the script but its easy practice to just make anti cheat that checks it and its a good way to get rid of hackers lowkey
-- now for an example i will show you a unarmed fist combat script 
-- it will use an upgraded version of the effectreplicator called statehandler 
-- i will not show it for security reason but you need to know that its priorty based 
-- meaning if attacking has a priorty of 0.5 and stun has a prioty of 0.9 
-- stun will remove attacking and overwrite it 


-- like the other shows diffrent aspects of scripting this one shows efficent line usage 
-- with using the connection like i did i can maximize what each line do 
-- achiving alot with minimal line making work pacing more efficent and fast 
-- its a server side and using suphi networking module to achive maxium speed and efficnetcy 

-- the hitbox module is similar to TSB where it gets targets from the client to deal with high moveing target and ping compensation 
-- ofc it also does sainty checks but its quite long and sepratated into 3 parts 

```
--Fist Server
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ServerStorage = game:GetService("ServerStorage")
local Debris = game:GetService("Debris")

local SharedModule = ReplicatedStorage.SharedModules
local Modules = ServerStorage.Modules

local Services = Modules.Services

local SharedHandlers = SharedModule.Handlers
local Handlers = Modules.Handlers

local StateHandler = require(SharedHandlers.StateHandler)
local AnimationHandler = require(SharedHandlers.AnimationHandler)
local VFXHandler = require(SharedHandlers.EffectsHandler)
local HitBoxHandler = require(Handlers.HitBoxHandler)
local CombatService = require(Services.CombatService)
local CoolDownHandler = require(Handlers.CoolDownHandler)

local AnimationFolder = ReplicatedStorage.Animations.Combat.Fist

local Swing : number = 1


local SlashWait = {
	0.25,
	0.25,
	0.25,
	0.25,
	0.25
}

local CoolDowns = {}

local Tool = {}


function Tool.Equipped(player : Player)

	local character = player.Character

end

function Tool.UnEquipped(player : Player)

	local character = player.Character

end

function Tool.Activated(player : Player,Addons)
	
	local character = player.Character
	
	local isAir = Addons[1]
	local isSprinting = Addons[2]
	
	local Humanoid: Humanoid = character:FindFirstChild("Humanoid")
	local HumanoidRootPart: HumanoidRootPart = character:FindFirstChild("HumanoidRootPart")

	if StateHandler:CheckStates(character,"Stun","Ragdoll","Block","EndLag", "Attacking") then
		return
	end


	local CurrentSwing = Swing

	-- Add Attacking State
	local Attacking = StateHandler:AddState(character, "Attacking")

	if not Attacking then
		return
	end
	
	if isAir and not CoolDownHandler:IsOnCooldown(character,"AirAttack") then
		return Tool.AirAttack(player,Attacking)
	end
	
	if isSprinting and not isAir then
		return Tool.SprintAttack(player,Attacking)
	end
	
	local EndLag = CurrentSwing == 5 and 0.5 or 0
	
	Attacking:Debris(0.5 + EndLag)
	
	-------------

	local AnimationTrack : AnimationTrack = AnimationHandler:PlayAnimation(character,AnimationFolder["Swing"..Swing])

	local HitConnection : RBXScriptConnection
	local interruptedConnections : {RBXScriptConnection} = {}
	local StopConnection : RBXScriptConnection

	
	HitConnection = AnimationTrack:GetMarkerReachedSignal("Hit"):Once(function()		
		local params : OverlapParams = OverlapParams.new()
		params.FilterType = Enum.RaycastFilterType.Exclude
		params.FilterDescendantsInstances = {character}

		local BoxData : HitBoxHandler.BoxInfo = {
			Base = HumanoidRootPart,
			CFrame = HumanoidRootPart.CFrame * CFrame.new(0,0,-2),
			Size =  Vector3.new(5,5,6),
			Visual = false,
			Color = Color3.fromRGB(255, 52, 55),
			Once = true,
			OverlapParams = params,
			Duration = 0.1,
		}
		HitBoxHandler.Box(BoxData,function(Model,Time)
			
			if not interruptedConnections then return end
			
			local TagData : CombatService.CombatData = {
				Attacker = character,
				Enemy = Model,
				Damage = 6,
				Type = "Fist",
				Swing = CurrentSwing,
				Stun = 0.9,
				NoPerfectParry = true,
			}
			if CurrentSwing == 5 then
				TagData.Damage = 8
				TagData.KnockBack = true
			end
			CombatService.TagHumanoid(TagData)
		end)
	end)

	StopConnection = AnimationTrack.Stopped:Once(function()
		HitConnection:Disconnect()
		StopConnection:Disconnect()
		for i , v in interruptedConnections do
			v:Disconnect()
		end
		HitConnection = nil
		StopConnection = nil
		interruptedConnections = nil
		
	end)
	
	task.defer(function()
		table.insert(interruptedConnections,StateHandler:GetStateSignal(character,"Stun"):Once(function(Method)
			AnimationTrack:Stop()
		end))
		table.insert(interruptedConnections,StateHandler:GetStateSignal(character,"AttackSkill"):Once(function(Method)
			AnimationTrack:Stop()
		end))	
	end)
	

	task.delay(SlashWait[CurrentSwing],function()
		
		if not StopConnection then
			return
		end
		
		VFXHandler.Run("FistSlash",{
			Player = player,
			Swing = CurrentSwing,
		})

		local Sound = ReplicatedStorage.Assets.Sounds.Fist.Swing:Clone()
		Sound.Parent = HumanoidRootPart
		Sound:Play()

		Debris:AddItem(Sound,3)
	end)

	Swing += 1
	if Swing > 5 then
		Swing = 1
	end

	task.spawn(function()
		local OldSwing = Swing
		task.wait(1)
		if Swing == OldSwing then
			Swing = 1
		end
	end)

end

function Tool.AirAttack(player : Player,Attacking)
	
	local character : Model = player.Character
	local HumanoidRootPart : BasePart = character.HumanoidRootPart
	
	CoolDownHandler.StartCoolDown(character,"AirAttack",3)
	Attacking:Debris(0.65)
	
	local AnimationTrack : AnimationTrack = AnimationHandler:PlayAnimation(character,AnimationFolder["AirAttack"]) 
	
	VFXHandler.RunOnly(player,"AerialMomentum",{Character = character})
	
	local HitConnection : RBXScriptConnection
	local StopConnection : RBXScriptConnection
	local interruptedConnection : RBXScriptConnection


	HitConnection = AnimationTrack:GetMarkerReachedSignal("Hit"):Once(function()
		local params : OverlapParams = OverlapParams.new()
		params.FilterType = Enum.RaycastFilterType.Exclude
		params.FilterDescendantsInstances = {character}

		local BoxData : HitBoxHandler.BoxInfo = {
			Base = HumanoidRootPart,
			CFrame = HumanoidRootPart.CFrame * CFrame.new(0,-1,-3),
			Size =  Vector3.new(5,5,6),
			Visual = false,
			Color = Color3.fromRGB(255, 52, 55),
			Once = true,
			OverlapParams = params,
			Duration = 0.25,
		}
		HitBoxHandler.Box(BoxData,function(Model,Time)
			local TagData : CombatService.CombatData = {
				Attacker = character,
				Enemy = Model,
				Damage = 6,
				Stun = 0.9,
				Type = "Fist",
			}
			
			CombatService.TagHumanoid(TagData)
		end)
	end)

	StopConnection = AnimationTrack.Stopped:Once(function()
		HitConnection:Disconnect()
		StopConnection:Disconnect()
		HitConnection = nil
		StopConnection = nil
	end)
	interruptedConnection = StateHandler:GetStateSignal(character,"Stun"):Once(function(Method)
		AnimationTrack:Stop()
	end)
end

function Tool.SprintAttack(player : Player,Attacking) 
	local character : Model = player.Character
	local HumanoidRootPart : BasePart = character.HumanoidRootPart

	CoolDownHandler.StartCoolDown(character,"AirAttack",3)
	Attacking:Debris(0.5)

	local AnimationTrack : AnimationTrack = AnimationHandler:PlayAnimation(character,AnimationFolder["RunningAttack"]) 
	
	VFXHandler.RunOnly(player,"SprintMomentum",{Character = character})

	local HitConnection : RBXScriptConnection
	local StopConnection : RBXScriptConnection
	local interruptedConnection : RBXScriptConnection


	HitConnection = AnimationTrack:GetMarkerReachedSignal("Hit"):Once(function()
		local params : OverlapParams = OverlapParams.new()
		params.FilterType = Enum.RaycastFilterType.Exclude
		params.FilterDescendantsInstances = {character}

		local BoxData : HitBoxHandler.BoxInfo = {
			Base = HumanoidRootPart,
			CFrame = HumanoidRootPart.CFrame * CFrame.new(0,0,-3.5),
			Size =  Vector3.new(5,5,7),
			Visual = false,
			Color = Color3.fromRGB(255, 52, 55),
			Once = true,
			OverlapParams = params,
			Duration = 0.2,
		}
		HitBoxHandler.Box(BoxData,function(Model,Time)
			local TagData : CombatService.CombatData = {
				Attacker = character,
				Enemy = Model,
				Damage = 6,
				Stun = 0.9,
				Type = "Fist",
			}

			CombatService.TagHumanoid(TagData)
		end)
	end)

	StopConnection = AnimationTrack.Stopped:Once(function()
		HitConnection:Disconnect()
		StopConnection:Disconnect()
		HitConnection = nil
		StopConnection = nil
	end)
	interruptedConnection = StateHandler:GetStateSignal(character,"Stun"):Once(function(Method)
		AnimationTrack:Stop()
	end)
end

local HasHit = {}

function Tool.Critical(player : Player,Addons)

	local character = player.Character

	local Humanoid: Humanoid = character:FindFirstChild("Humanoid")
	local HumanoidRootPart: BasePart = character:FindFirstChild("HumanoidRootPart")
	
	if CoolDownHandler:IsOnCooldown(character,"Critical") then
		return
	end
	
	local Attacking = StateHandler:AddState(character, "Attacking",1)

	if not Attacking then
		return
	end
	
	CoolDownHandler.StartCoolDown(character,"Critical",1.5)
	
	Attacking:Debris(0.8)
	
	VFXHandler.Run("Highlight",{
		Target = character,
		FillColor = Color3.fromRGB(255, 75, 78),
		FadeTime = 1
	})
	
	VFXHandler.Run("Rings",{
		Target = character,
	})
	
	local Sound = ReplicatedStorage.Assets.Sounds.Fist.CriticalStartUp:Clone()
	Sound.Parent = HumanoidRootPart
	Sound:Play()
	Debris:AddItem(Sound,3)

	local AnimationTrack : AnimationTrack = AnimationHandler:PlayAnimation(character,AnimationFolder["Critical"])

	local HitConnection : RBXScriptConnection
	local interruptedConnection : RBXScriptConnection
	local StopConnection : RBXScriptConnection

	HitConnection = AnimationTrack:GetMarkerReachedSignal("Hit"):Once(function()
		
		VFXHandler.Run("FistCrit",{
			Target = character,
		})
		
		local params : OverlapParams = OverlapParams.new()
		params.FilterType = Enum.RaycastFilterType.Exclude
		params.FilterDescendantsInstances = {character}

		local BoxData : HitBoxHandler.BoxInfo = {
			Base = HumanoidRootPart,
			CFrame = HumanoidRootPart.CFrame * CFrame.new(0,0,-4),
			Size =  Vector3.new(5,5,8),
			Visual = false,
			Color = Color3.fromRGB(255, 52, 55),
			Once = true,
			OverlapParams = params,
			Duration = 0.1,
		}
		HitBoxHandler.Box(BoxData,function(Model,Time)

			if not interruptedConnection then return end

			local TagData : CombatService.CombatData = {
				Attacker = character,
				Enemy = Model,
				Damage = 15,
				Type = "Fist",
				Stun = 0.9,
				BlockBreak = true,
			}
			
			CombatService.TagHumanoid(TagData)
		end)
	end)

	StopConnection = AnimationTrack.Stopped:Once(function()
		HitConnection:Disconnect()
		StopConnection:Disconnect()
		interruptedConnection:Disconnect()
		HitConnection = nil
		StopConnection = nil
		interruptedConnection = nil

	end)

	interruptedConnection = StateHandler:GetStateSignal(character,"Stun"):Once(function(Method)
		AnimationTrack:Stop()
	end)

end

function Tool.Block(player : Player,Bool)
	local character = player.Character

	if not Bool then
		StateHandler.RemoveClass(character,"Block")
		AnimationHandler:StopAnimation(character,AnimationFolder.Block)
		return
	end
	
	if CoolDownHandler:IsOnCooldown(character,"Block") then
		return
	end
	
	local Parry = StateHandler:AddState(character,"Parry")
	local PerfectParry = StateHandler:AddState(character,"PerfectParry")
	Parry:Debris(0.3)
	PerfectParry:Debris(0.1)
	CoolDownHandler.StartCoolDown(character,"Block",2)
	local preBlock : AnimationTrack = AnimationHandler:PlayAnimation(character,AnimationFolder.PreParry)

end


return Tool

```


