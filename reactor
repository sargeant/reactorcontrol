-- Automatic control program for Big Reactors by ErogenousBeef – http://big-reactors.com/
-- Author: Sam Sargeant <sam@sargeant.net.nz>

-- Originally based on http://pastebin.com/V0eueruY by Emily Backes <lucca@accela.net>
-- Adapted for use with OpenComputers

-- Copyright (c) 2014, Sam Sargeant <sam@sargeant.net.nz>
-- All rights reserved.

-- Redistribution and use in source and binary forms, with or without
-- modification, are permitted provided that the following conditions are met:
--     * Redistributions of source code must retain the above copyright
--       notice, this list of conditions and the following disclaimer.
--     * Redistributions in binary form must reproduce the above copyright
--       notice, this list of conditions and the following disclaimer in the
--       documentation and/or other materials provided with the distribution.
--     * Neither the name of the <organization> nor the
--       names of its contributors may be used to endorse or promote products
--       derived from this software without specific prior written permission.

-- THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
-- ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
-- WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
-- DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER BE LIABLE FOR ANY
-- DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
-- (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
-- LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
-- ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
-- (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
-- SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

local component = require("component")
local serialization = require("serialization")

local modem = component.modem
local portNumber = 2805

local term = require("term")

-- Max energy in a reactor's internal cell
local emax=10000000

-- wrap everything in an exception handler
local ok,msg=pcall(function ()
local r
local m
local redirected=false
local p

function setupDevs()
  r=assert(component.br_reactor)
  if (not r.getConnected()) then
    return nil, "Computer port not connected to a valid reactor"
  end
  --if (r.getNumberOfControlRods() <1) then
  --  return nil, "Reactor seems invalid"
  --end
  r.getEnergyPercent = function ()
    return math.floor(1000 * r.getEnergyStored() / emax)/10
  end
  if r.nativeEPLT then
    r.getEnergyProducedLastTick = r.nativeEPLT
  end
  r.nativeEPLT = r.getEnergyProducedLastTick
  r.getEnergyProducedLastTick = function ()
    return math.floor(r.nativeEPLT()*1000)/1000
  end

  term.setCursorBlink(false)
end

function ft ()
  -- local d=os.day()
  -- local t=os.time()
  -- local h=math.floor(t)
  -- local m=math.floor((t-h)*60)
  -- return string.format("Day %d, %02d:%02d",d,h,m)
     return "timestamp..."
end

function log (msg)
  local stamp=ft()
  print (stamp..": "..msg)
end

function tableWidth(t)
  local width=0
  for _,v in pairs(t) do
    if #v>width then width=#v end
  end
  return width + 4
end

function ljust(s,w)
  local pad=w-#s
  return s .. string.rep(" ",pad)
end

function rjust(s,w)
  local pad=w-#s
  return string.rep(" ",pad) .. s
end

function display()
  local floatUnits = {"%","C","mB","RF/t"}
  term.clear()
  term.setCursor(1,1)
  msg = {name="Reactor A - TEST"}
  msg["metric"] = {}
  print("Reactor Status")
  print("")
  local funcs={"Connected","Active","NumberOfControlRods","EnergyStored","EnergyPercent","FuelTemperature","CasingTemperature","FuelAmount","WasteAmount","FuelAmountMax", "FuelReactivity", "FuelConsumedLastTick", "EnergyProducedLastTick"}
  local units={"","","","RF","%","C","C","mB","mB","mB","%","mB", "RF/t"}
  local values={}
  for _,v in pairs(funcs) do
    thisVal = tostring(r["get"..v]())
    values[#values+1] = thisVal
    msg["metric"][v] = thisVal
  end

  local funcW=tableWidth(funcs)
  local valW=tableWidth(values)
  for i,v in pairs(funcs) do
    print(string.format("%-12s: %s %s", v, values[i], units[i]))
  end

  modem.broadcast(2805, serialization.serialize(msg))
end

log("Starting")
setupDevs()
while true do
  local e=r.getEnergyStored()
  local p=math.floor(100*e/emax)
  local a=p<100
  local elt=r.getEnergyProducedLastTick()
  display()
  r.setAllControlRodLevels(p)
  r.setActive(a)
  os.sleep(1)
  -- local event,p1,p2,p3,p4,p5 = os.pullEvent()
  -- if event == "key" then
  --   break
  -- elseif event == "peripheral_detach" or event == "peripheral" or event == "monitor_resize" then
  --   setupDevs()
  -- elseif not (event == "timer" or event=="disk" or event=="disk_eject") then
  --   error("received "..event)
  -- end
end

end)

error(msg)