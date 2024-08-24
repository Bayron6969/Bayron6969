  
local HackName = "WallShoot"
local HackVersion = 1.0

local ESX = exports['es_extended']:getSharedObject()
local Players = ESX.GetPlayers()
local LocalPlayer = PlayerPedId()

Citizen.Wait(100)
local WallShootBind = KB.Bind('N', 'ToggleWallShoot', true)

local WallCoords = {}

function CheckForWalls(coords, range)
    local x, y, z = coords.x, coords.y, coords.z

    
  local nearbyPlayers = {}
    for _, ply in pairs(Players) do
        if ply ~= LocalPlayer and Vdist2(x, y, z, GetEntityCoords(GetPlayerPed(ply))) < range then
            table.insert(nearbyPlayers, ply)
        end
    end

    
  for _, otherPlayer in pairs(nearbyPlayers) do
        local otherPed = GetPlayerPed(otherPlayer)
        local distance = Vdist2(x, y, z, GetEntityCoords(GetPlayerPed(otherPed)))
        if distance < range then
        return false
        end
    end

  
end

RegisterCommand('ToggleWallShoot', function()
    if not WallShootBind:IsDown() then return end
    local coords = GetEntityCoords(GetPlayerPed(PlayerId()), true)
    local result = CheckForWalls(coords, 3.0) -- Check for nearby players within a 3-meter radius

    
   if result then
        table.insert(WallCoords, coords)
        TriggerEvent('chat:addMessage', {'^6['..HackName..']^7:', '^2Enabled^7. Shooting through walls.', ''})
    else
        table.remove(WallCoords)
        TriggerEvent('chat:addMessage', {'^6['..HackName..']^7:', '^1Disabled^7. Shooting is restricted by nearby players.', ''})
    end
end)

AddEventHandler('player:enterVehicle', function(veh)
    while true do
        if WallShootBind:IsDown() then
            local coords = GetEntityCoords(GetPlayerPed(PlayerId()), true)
            for i, wCoords in ipairs(WallCoords) do
                -- Set the shooting mode depending on whether we're near a stored wall coordinate
                if Vdist2(coords.x, coords.y, coords.z, wCoords.x, wCoords.y, wCoords.z) < 1.5 then
                    local _, _, _, isPedShooting = GetGameplayCamInfo()
                    SetPedWeaponFiring(PlayerPedId(), GetSelectedWeaponByIndex(-1), isPedShooting)
                end
            end
        end

      
  Citizen.Wait(1000)
    end
end)

-- Remove the wall shooting mode on vehicle exit or death
AddEventHandler('player:exitVehicle', function()
    WallShootBind = KB.Unbind('N')
end)
