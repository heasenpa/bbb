game.Players.LocalPlayer.Idled:Connect(function()
    if active then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)
