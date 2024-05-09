
# *Update Sip Extenstion Status in Database*


*Update Lua Module Config File*
###### *File PATH:- /usr/local/freeswitch/conf/autoload_configs/lua.conf.xml*
Add This line In lua.conf.xml file
```bash
  <hook event="CUSTOM" subclass="sofia::unregister" script="unregister.lua"/>
  <hook event="CUSTOM" subclass="sofia::register" script="register.lua"/>
```

#### Create register.lua file  in /usr/local/freeswitch/scripts 
###### *File PATH:- /usr/local/freeswitch/scripts/register.lua*

```bash
---@diagnostic disable: undefined-global
package.path = package.path .. ";" .. [[/usr/share/lua/5.3/?.lua]] .. ";" .. [[/usr/local/share/lua/5.3/?.lua]] .. ";"
local script_path = "/usr/local/freeswitch/scripts/";
dofile(script_path .. "utils-db.lua")

-- register.lua

-- Retrieve and log specific headers associated with the event
local user_agent = event:getHeader("User-Agent")
local contact = event:getHeader("Contact")
local status = event:getHeader("status")
local username = event:getHeader("username")

-- Log event data
freeswitch.consoleLog("info", "REGISTER Event - User Agent: " .. (user_agent or "N/A") .. "\n")
freeswitch.consoleLog("info", "REGISTER Event - Contact: " .. (contact or "N/A") .. "\n")
freeswitch.consoleLog("info", "REGISTER Event - Contact: " .. (status or "N/A") .. "\n")
freeswitch.consoleLog("info", "REGISTER Event - Contact: " .. (username or "N/A") .. "\n")

local data={
    user_agent=user_agent,
    status=status,
    username=username
}
registerExtention(data)
```

#### Create unregister.lua file  in /usr/local/freeswitch/scripts 
###### *File PATH:- /usr/local/freeswitch/scripts/unregister.lua*

```bash
---@diagnostic disable: undefined-global
package.path = package.path .. ";" .. [[/usr/share/lua/5.3/?.lua]] .. ";" .. [[/usr/local/share/lua/5.3/?.lua]] .. ";"

local script_path = "/usr/local/freeswitch/scripts/";
dofile(script_path .. "utils-db.lua")

-- Retrieve and log all session variables
freeswitch.consoleLog("console","diagnosticr UnRegister session variables")


-- unregister.lua

-- Retrieve and log specific headers associated with the event
local user_agent = event:getHeader("User-Agent")
local contact = event:getHeader("Contact")
local username = event:getHeader("username")

-- Log event data
freeswitch.consoleLog("info", "UnRegister Event - User Agent: " .. (user_agent or "N/A") .. "\n")
freeswitch.consoleLog("info", "UnRegister Event - Contact: " .. (contact or "N/A") .. "\n")
freeswitch.consoleLog("info", "UnRegister Event - Contact: " .. (username or "N/A") .. "\n")

unregisterExtention(username)
```

#### Create Common File utils-db.lua in /usr/local/freeswitch/scripts
###### *File PATH:- /usr/local/freeswitch/scripts/utils-db.lua*

For All Db Related Function Create in This File

#### Connect  Mongo Db Database 
[Install  Lua-mongo](lua-mongo.md) 

#### *Connect Database*
```bash
package.path = package.path .. ";" .. [[/usr/share/lua/5.3/?.lua]] .. ";" .. [[/usr/local/share/lua/5.3/?.lua]] .. ";"
local mongo = require "mongo"
local client = mongo.Client('your_databse_url')
local database = "database"
local ExtensionModel = client:getCollection(database, "extensions") //extensions is collection name
```

```bash
--User Extension Register Status Update
function registerExtention(data)
  freeswitch.consoleLog("console", "Register Status Update data" .. inspect(data))
  -- Define the filter to match documents
  local filter = { username = data.username }

  local update = {
    ["$set"] = {
    user_agent = data.user_agent,
    isRegister = 1,
    register_status = data.status,
    register_date = os.time() * 1000,
    }
  }

  -- Execute the update query
  local result = ExtensionModel:updateOne(filter, update)
  return result
end

--User Extension Unregister Status Update
function unregisterExtention(username)
  -- Define the filter to match documents
  local filter = { username = username }

  local update = {
    ["$set"] = {
      user_agent = "",
      isRegister = 0,
      register_status = "Unregister",
    }
  }
  -- Execute the update query
  local result = ExtensionModel:updateOne(filter, update )
  return result
end

```



