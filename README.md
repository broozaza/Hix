-- ฟังก์ชันแปลงเลขเป็นภาษาไทย (เหมือนเดิม)
local function numberToThai(n)
    local thaiNumbers = {[0] = "ศูนย์", [1] = "หนึ่ง", [2] = "สอง", [3] = "สาม", [4] = "สี่", [5] = "ห้า", [6] = "หก", [7] = "เจ็ด", [8] = "แปด", [9] = "เก้า"}
    local thaiUnits = {"", "สิบ", "ร้อย", "พัน"}
    local thaiText = ""
    local digits = tostring(n):reverse()

    for i = 1, #digits do
        local digit = tonumber(digits:sub(i, i))
        local unit = thaiUnits[i]
        
        if digit == 1 and i == 2 then
            thaiText = "สิบ" .. thaiText
        elseif digit == 2 and i == 2 then
            thaiText = "ยี่" .. unit .. thaiText
        elseif digit > 0 then
            thaiText = thaiNumbers[digit] .. unit .. thaiText
        end
    end
    
    return thaiText
end

-- ตัวแปร
local delayTime = 2
local maxCount = 1000
local count = 1
local running = false

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- สร้าง GUI เมนู
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

-- ปุ่มเปิด/ปิด UI
local toggleMenuButton = Instance.new("TextButton")
toggleMenuButton.Size = UDim2.new(0, 120, 0, 50)
toggleMenuButton.Position = UDim2.new(0, 10, 0, 10)
toggleMenuButton.Text = "Show Menu"
toggleMenuButton.BackgroundColor3 = Color3.fromRGB(0, 20, 40)
toggleMenuButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleMenuButton.Font = Enum.Font.SourceSansBold
toggleMenuButton.TextSize = 18
toggleMenuButton.Parent = screenGui

-- Frame ของเมนู พร้อมการเลื่อน
local menuFrame = Instance.new("ScrollingFrame")
menuFrame.Size = UDim2.new(0, 300, 0, 200)
menuFrame.Position = UDim2.new(0.5, -150, 0.5, -100)
menuFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
menuFrame.BorderSizePixel = 0
menuFrame.Visible = false
menuFrame.ScrollBarThickness = 8
menuFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 100, 150)
menuFrame.Parent = screenGui

-- ปุ่มเปิด/ปิดการทำงาน (Toggle)
local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 200, 0, 50)
toggleButton.Position = UDim2.new(0, 50, 0, 20)
toggleButton.Text = "Start"
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 120, 220)
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 18
toggleButton.Parent = menuFrame

-- ปุ่มรีเซ็ตการนับ
local resetButton = Instance.new("TextButton")
resetButton.Size = UDim2.new(0, 200, 0, 50)
resetButton.Position = UDim2.new(0, 50, 0, 80)
resetButton.Text = "Reset Counter"
resetButton.BackgroundColor3 = Color3.fromRGB(0, 80, 180)
resetButton.TextColor3 = Color3.fromRGB(255, 255, 255)
resetButton.Font = Enum.Font.SourceSansBold
resetButton.TextSize = 18
resetButton.Parent = menuFrame

-- ช่องใส่จำนวนสูงสุดที่ต้องการนับ
local maxCountBox = Instance.new("TextBox")
maxCountBox.Size = UDim2.new(0, 200, 0, 50)
maxCountBox.Position = UDim2.new(0, 50, 0, 140)
maxCountBox.Text = tostring(maxCount)
maxCountBox.PlaceholderText = "Enter Max Count"
maxCountBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
maxCountBox.TextColor3 = Color3.fromRGB(0, 0, 0)
maxCountBox.Font = Enum.Font.SourceSans
maxCountBox.TextSize = 16
maxCountBox.Parent = menuFrame

-- ฟังก์ชัน jjsCounterAndJump (เหมือนเดิม)
local function jjsCounterAndJump()
    while count <= maxCount and running do
        local thaiNumber = numberToThai(count)
        
        -- ส่งข้อความในช่องแชท
        ReplicatedStorage.DefaultChatSystemChatEvents.SayMessageRequest:FireServer(thaiNumber, "All")
        
        -- กระโดด
        if character and character:FindFirstChild("Humanoid") then
            local humanoid = character:FindFirstChild("Humanoid")
            if humanoid then
                humanoid:Move(Vector3.new(0, 1, 0))
                humanoid.Jump = true
            end
        end
        
        count = count + 1
        wait(delayTime)
    end
end

-- ฟังก์ชันเปิด/ปิดการทำงาน
local function toggleScript()
    running = not running
    toggleButton.Text = running and "Stop" or "Start"
    if running then
        jjsCounterAndJump()
    end
end

-- ฟังก์ชันรีเซ็ตการนับ
local function resetCounter()
    count = 1
end

-- ฟังก์ชันอัปเดตจำนวนสูงสุด
local function updateMaxCount()
    local newMax = tonumber(maxCountBox.Text)
    if newMax then
        maxCount = newMax
    else
        maxCountBox.Text = tostring(maxCount)
    end
end

-- ฟังก์ชันแสดง/ซ่อนเมนู
local function toggleMenuVisibility()
    menuFrame.Visible = not menuFrame.Visible
    toggleMenuButton.Text = menuFrame.Visible and "Hide Menu" or "Show Menu"
end

-- เชื่อมโยงปุ่ม
toggleButton.MouseButton1Click:Connect(toggleScript)
resetButton.MouseButton1Click:Connect(resetCounter)
maxCountBox.FocusLost:Connect(updateMaxCount)
toggleMenuButton.MouseButton1Click:Connect(toggleMenuVisibility)
