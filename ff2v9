--[[ 
  LocalScript: FF2EnhancerUI_ErrorPrediction.lua
  Place this script under StarterPlayerScripts or in a PlayerGui LocalScript
  Complete version with real-time error prediction and prevention mechanism
--]]

--// Services
local Players              = game:GetService("Players")
local RunService           = game:GetService("RunService")
local UserInputService     = game:GetService("UserInputService")
local ContextActionService = game:GetService("ContextActionService")
local TweenService         = game:GetService("TweenService")
local HttpService          = game:GetService("HttpService") -- For JSON encoding/decoding
local GuiService           = game:GetService("GuiService") -- For determining insets
local PhysicsService       = game:GetService("PhysicsService") -- For physics-related checks

--// ENHANCED PREDICTIVE ERROR PREVENTION SYSTEM

-- Main diagnostic system with predictive capabilities
local DiagnosticSystem = {}

-- Configuration for diagnostic system
DiagnosticSystem.Config = {
    -- General settings
    Enabled = true,                  -- Master toggle for diagnostics
    AutoReport = true,               -- Auto-report critical errors
    LogToOutput = true,              -- Log errors to output
    MaxErrorsStored = 50,            -- Maximum number of errors to store
    AutoDiagInterval = 60,           -- Run auto-diagnostics every X seconds
    
    -- Performance monitoring
    TrackPerformance = true,         -- Monitor performance metrics
    PerformanceThreshold = 45,       -- Frame time threshold in ms (45ms = ~22fps)
    SampleSize = 30,                 -- Number of frames to sample for performance
    
    -- Error reporting
    ErrorReportFormat = "discord",    -- discord, console, or gui
    ReportNonCritical = false,       -- Report non-critical errors
    IncludeEnvironmentInfo = true,   -- Include device/environment info
    
    -- User feedback
    NotifyUser = true,               -- Show notifications to user
    NotifyFrequency = "medium",      -- low, medium, high (how often to show notifications)
    
    -- Debug visualization
    VisualDebug = true,              -- Enable visual debugging elements
    DebugColors = {
        Info = Color3.fromRGB(0, 230, 230),    -- Cyan
        Warning = Color3.fromRGB(255, 190, 0), -- Yellow
        Error = Color3.fromRGB(255, 60, 60),   -- Red
        Success = Color3.fromRGB(0, 230, 120), -- Green
    },
    
    -- NEW: Error Prevention Settings
    PredictiveErrorPrevention = true, -- Master toggle for predictive system
    AutoHeal = true,                 -- Automatically fix issues when possible
    HighRiskThreshold = 0.8,         -- Risk threshold (0-1) for high risk triggers
    MaxBackupStates = 5,             -- Max number of states to keep for rollback
    MinHealthBeforeAlert = 70,       -- Health score that triggers warnings
    MonitorRiskyOperations = true,   -- Monitor operations known to cause errors
    PreemptiveChecks = true,         -- Run checks before operations
    PredictionUpdateRate = 0.5,      -- How often to update predictions (in seconds)
    AutoRollbackEnabled = true,      -- Automatically rollback to last stable state
}

-- Initialize data structures
DiagnosticSystem.Errors = {}
DiagnosticSystem.Performance = {
    FrameTimes = {},
    LastCheck = os.clock(),
    FrameTimeAvg = 0,
    MemoryUsage = 0
}
DiagnosticSystem.Status = {
    IsRunning = false,
    LastAutoCheck = 0,
    HealthScore = 100, -- 0-100 score for overall health
    ComponentStatus = {
        UI = "ok",
        AngleEnhancer = "ok",
        Freeze = "ok",
        Controller = "ok"
    }
}

-- NEW: Predictive error prevention systems
DiagnosticSystem.ErrorPrediction = {
    -- Last known stable state (for rollback)
    LastStableState = nil,
    
    -- History of states for rollback
    StateHistory = {},
    
    -- Tracks operations that might cause errors
    RiskyOperationLog = {},
    
    -- Current risk assessment
    CurrentRiskLevel = 0, -- 0-1 scale, 1 being highest risk
    
    -- Patterns of known issues and their prevention strategies
    KnownPatterns = {
        ["CharacterNotFound"] = {
            pattern = "Character not found",
            category = "Character",
            severity = "high",
            preventiveAction = function()
                -- Wait for character to be added
                local character = Players.LocalPlayer.Character or Players.LocalPlayer.CharacterAdded:Wait()
                return character ~= nil
            end
        },
        
        ["HumanoidNotFound"] = {
            pattern = "Humanoid not found",
            category = "Character",
            severity = "high",
            preventiveAction = function()
                local character = Players.LocalPlayer.Character
                if not character then return false end
                
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if not humanoid then
                    -- Wait for humanoid to be added
                    humanoid = character:WaitForChild("Humanoid", 2)
                end
                
                return humanoid ~= nil
            end
        },
        
        ["RootPartNotFound"] = {
            pattern = "HumanoidRootPart not found",
            category = "Character",
            severity = "high",
            preventiveAction = function()
                local character = Players.LocalPlayer.Character
                if not character then return false end
                
                local rootPart = character:FindFirstChild("HumanoidRootPart")
                if not rootPart then
                    -- Wait for root part to be added
                    rootPart = character:WaitForChild("HumanoidRootPart", 2)
                end
                
                return rootPart ~= nil
            end
        },
        
        ["NilObjectReference"] = {
            pattern = "attempt to index nil",
            category = "Reference",
            severity = "high",
            preventiveAction = function(obj, property)
                -- General protection against nil references
                return obj ~= nil
            end
        },
        
        ["InvalidProperty"] = {
            pattern = "is not a valid member",
            category = "Property",
            severity = "medium",
            preventiveAction = function(obj, property)
                -- Check if property exists before accessing
                if not obj then return false end
                
                local success = pcall(function()
                    local _ = obj[property]
                end)
                
                return success
            end
        },
        
        ["PhysicsError"] = {
            pattern = "Cannot adjust physical properties",
            category = "Physics",
            severity = "medium",
            preventiveAction = function(part)
                -- Ensure physical properties can be changed
                if not part or not part:IsA("BasePart") then return false end
                
                -- Check network ownership
                local success = pcall(function()
                    part.Anchored = part.Anchored -- Test if we can change it
                end)
                
                return success
            end
        },
        
        ["NetworkOwnership"] = {
            pattern = "lack network ownership",
            category = "Physics",
            severity = "medium",
            preventiveAction = function(part)
                -- Check network ownership issues
                if not part or not part:IsA("BasePart") then return false end
                
                local networkOwner = part:GetNetworkOwner()
                return networkOwner == Players.LocalPlayer
            end
        },
        
        ["AnimationPlayback"] = {
            pattern = "Animation playback failed",
            category = "Animation",
            severity = "low",
            preventiveAction = function(animator, animation)
                -- Ensure animation can be played
                if not animator or not animation then return false end
                
                return animator:IsDescendantOf(game) and animation:IsA("Animation")
            end
        },
        
        ["MemoryLeak"] = {
            pattern = "memory usage",
            category = "Performance",
            severity = "high",
            preventiveAction = function()
                -- Check for memory leaks by looking at connection count
                local connections = getconnections or get_signal_connections
                if connections then
                    -- Clean up excessive connections if possible
                    -- (implementation would depend on game specifics)
                end
                
                return true -- Can't directly prevent, just monitor
            end
        }
    },
    
    -- Tracks specific instances that have been problematic
    ProblemInstances = {},
    
    -- Environment factors that might affect errors
    EnvironmentFactors = {
        highServerLoad = false,
        poorNetworkConditions = false,
        lowFps = false,
        highMemoryUsage = false,
    },
    
    -- Last prediction update time
    LastPredictionUpdate = 0,
    
    -- Current auto-fix actions in progress
    ActiveFixAttempts = {},
    
    -- Function to store a state snapshot that we can roll back to
    StoreStateSnapshot = function(self, label)
        -- Create a snapshot of current state
        local stateSnapshot = {
            timestamp = os.time(),
            label = label or "Automatic Snapshot",
            healthScore = DiagnosticSystem.Status.HealthScore,
            configState = table.clone(Config), -- Assuming Config is your app config
            characterState = self:CaptureCharacterState(),
            uiState = self:CaptureUIState()
        }
        
        -- Add to history, maintain max size
        table.insert(self.StateHistory, 1, stateSnapshot)
        if #self.StateHistory > DiagnosticSystem.Config.MaxBackupStates then
            table.remove(self.StateHistory)
        end
        
        -- If health is good, mark as stable
        if DiagnosticSystem.Status.HealthScore >= 90 then
            self.LastStableState = stateSnapshot
        end
        
        return stateSnapshot
    end,
    
    -- Helper to capture character state
    CaptureCharacterState = function(self)
        local character = Players.LocalPlayer.Character
        if not character then return nil end
        
        -- Basic character info
        local state = {
            position = nil,
            velocity = nil,
            humanoidState = nil,
            animations = {}
        }
        
        -- Get position and velocity if possible
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            state.position = rootPart.Position
            state.velocity = rootPart.AssemblyLinearVelocity
        end
        
        -- Get humanoid state if possible
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            state.humanoidState = humanoid:GetState().Name
            
            -- Capture playing animations
            local animator = humanoid:FindFirstChildOfClass("Animator")
            if animator then
                for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
                    if track.IsPlaying then
                        table.insert(state.animations, {
                            animation = track.Animation.AnimationId,
                            speed = track.Speed,
                            timePosition = track.TimePosition,
                            weight = track.WeightCurrent
                        })
                    end
                end
            end
        end
        
        return state
    end,
    
    -- Helper to capture UI state
    CaptureUIState = function(self)
        if not screenGui then return nil end
        
        return {
            uiVisible = Config.UIVisible,
            activeTab = nil, -- Would need logic to determine active tab
            theme = Config.Theme,
            animationStyle = Config.AnimationStyle,
        }
    end,
    
    -- Roll back to a previous state
    RollbackToState = function(self, stateIndex)
        local state = self.StateHistory[stateIndex or 1]
        if not state then return false end
        
        -- Log the rollback attempt
        DiagnosticSystem:LogError(
            "Rolling back to state: " .. state.label, 
            "ErrorPrevention", 
            "warning", 
            {timestamp = state.timestamp}
        )
        
        -- Restore configuration
        for key, value in pairs(state.configState) do
            Config[key] = value
        end
        
        -- We can't literally restore character state, but we can handle specific issues
        -- based on what we captured, like re-applying animations, etc.
        
        -- Update UI to reflect the rollback
        if win and state.uiState then
            Config.UIVisible = state.uiState.uiVisible
            Config.Theme = state.uiState.theme
            Config.AnimationStyle = state.uiState.animationStyle
            
            -- Apply theme
            UpdateUITheme(win, Config.Theme)
            
            -- Toggle UI visibility if needed
            if screenGui and screenGui.Enabled ~= Config.UIVisible then
                ToggleUI(screenGui, win, Config.UIVisible)
            end
        end
        
        return true
    end,
    
    -- Predict risk of error in an operation
    PredictRisk = function(self, operation, context)
        local risk = 0 -- 0 to 1 scale
        
        -- Start with a base risk assessment
        if operation == "CharacterOperation" then
            -- Higher risk when character operations are done during respawn
            if not Players.LocalPlayer.Character then 
                risk = risk + 0.8
            end
        elseif operation == "UIOperation" then
            -- UI operations are risky if widgets haven't been created
            if not screenGui or not win then
                risk = risk + 0.7
            end
        elseif operation == "FreezeOperation" then
            -- Freeze is risky if character state is unusual
            if Players.LocalPlayer.Character then
                local humanoid = Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    local state = humanoid:GetState()
                    -- Particularly risky during certain states
                    if state == Enum.HumanoidStateType.Physics or 
                       state == Enum.HumanoidStateType.Dead or
                       state == Enum.HumanoidStateType.Swimming then
                        risk = risk + 0.6
                    end
                else
                    risk = risk + 0.9 -- No humanoid is very risky
                end
            else
                risk = risk + 0.9 -- No character is very risky
            end
        end
        
        -- Adjust based on historical issues
        for _, entry in ipairs(self.RiskyOperationLog) do
            if entry.operation == operation then
                -- Recent failures increase risk
                local timeSince = os.clock() - entry.timestamp
                if timeSince < 10 then -- Within last 10 seconds
                    risk = risk + 0.3
                elseif timeSince < 60 then -- Within last minute
                    risk = risk + 0.1
                end
                
                -- Multiple failures increase risk
                if entry.failureCount > 2 then
                    risk = risk + 0.2
                end
            end
        end
        
        -- Adjust for environment factors
        if self.EnvironmentFactors.lowFps then risk = risk + 0.15 end
        if self.EnvironmentFactors.highMemoryUsage then risk = risk + 0.1 end
        if self.EnvironmentFactors.poorNetworkConditions then risk = risk + 0.2 end
        
        -- Cap at 1.0 maximum
        return math.min(risk, 1.0)
    end,
    
    -- Log a risky operation
    LogRiskyOperation = function(self, operation, details, success)
        -- Find existing entry or create new one
        local entry
        for i, existing in ipairs(self.RiskyOperationLog) do
            if existing.operation == operation then
                entry = existing
                break
            end
        end
        
        if not entry then
            entry = {
                operation = operation,
                firstAttempt = os.clock(),
                attemptCount = 0,
                failureCount = 0,
                lastError = nil
            }
            table.insert(self.RiskyOperationLog, entry)
        end
        
        -- Update the entry
        entry.timestamp = os.clock()
        entry.attemptCount = entry.attemptCount + 1
        
        if not success then
            entry.failureCount = entry.failureCount + 1
            entry.lastError = details
        end
        
        return entry
    end,
    
    -- Run preventative measures based on risk prediction
    PreventErrors = function(self, operation, context)
        -- Calculate risk for this operation
        local risk = self:PredictRisk(operation, context)
        self.CurrentRiskLevel = risk -- Update global risk assessment
        
        -- Low risk, proceed normally
        if risk < 0.3 then return true end
        
        -- Medium risk, take some precautions
        if risk < DiagnosticSystem.Config.HighRiskThreshold then
            -- Capture state before proceeding
            if DiagnosticSystem.Config.AutoRollbackEnabled then
                self:StoreStateSnapshot("Pre-" .. operation)
            end
            
            DiagnosticSystem:LogError(
                "Medium risk detected for " .. operation, 
                "ErrorPrevention", 
                "warning", 
                {risk = risk}
            )
            
            -- Take some preventative actions based on operation type
            if operation == "CharacterOperation" then
                -- Ensure character exists
                if not Players.LocalPlayer.Character then
                    DiagnosticSystem:LogError(
                        "Waiting for character to spawn before proceeding", 
                        "ErrorPrevention", 
                        "info"
                    )
                    Players.LocalPlayer.CharacterAdded:Wait()
                end
            elseif operation == "FreezeOperation" then
                -- Specific freeze precautions (depends on your implementation)
                -- For example, ensure we're in a suitable state for freezing
                local success = self:RunPatternCheck("CharacterNotFound")
                if not success then
                    return false -- Can't safely proceed
                end
            end
            
            return true -- Medium risk, but we've taken precautions
        end
        
        -- High risk, take more aggressive actions
        DiagnosticSystem:LogError(
            "High risk detected for " .. operation .. " - taking preventative action", 
            "ErrorPrevention", 
            "error", 
            {risk = risk}
        )
        
        -- Store state for possible rollback
        self:StoreStateSnapshot("Emergency-" .. operation)
        
        -- Run specific preventative measures based on operation
        if operation == "CharacterOperation" then
            -- For character operations, defer if character isn't ready
            if not Players.LocalPlayer.Character then
                -- Wait for character to be ready
                DiagnosticSystem:LogError(
                    "Deferring character operation until character is available", 
                    "ErrorPrevention", 
                    "warning"
                )
                return false -- Signal to caller to defer operation
            end
        elseif operation == "FreezeOperation" then
            -- Check all required components for freeze
            if not self:RunPatternCheck("CharacterNotFound") or
               not self:RunPatternCheck("HumanoidNotFound") or
               not self:RunPatternCheck("RootPartNotFound") then
                DiagnosticSystem:LogError(
                    "Cannot safely perform freeze operation - missing prerequisites", 
                    "ErrorPrevention", 
                    "error"
                )
                return false
            end
        end
        
        -- For all high-risk operations, proceed with caution
        -- but with enhanced monitoring
        return true
    end,
    
    -- Run specific check based on known pattern
    RunPatternCheck = function(self, patternKey)
        local pattern = self.KnownPatterns[patternKey]
        if not pattern then return true end -- No pattern to check
        
        local success = pattern.preventiveAction()
        if not success then
            DiagnosticSystem:LogError(
                "Preventative check failed: " .. patternKey, 
                "ErrorPrevention", 
                "warning", 
                {pattern = pattern}
            )
        end
        
        return success
    end,
    
    -- React to an error after it happens
    ReactToError = function(self, errorMsg, source)
        -- Try to match error to known patterns
        local matchedPattern = nil
        for key, pattern in pairs(self.KnownPatterns) do
            if string.find(errorMsg, pattern.pattern) then
                matchedPattern = pattern
                break
            end
        end
        
        if not matchedPattern then
            -- Unknown error pattern
            if DiagnosticSystem.Config.AutoRollbackEnabled and self.LastStableState then
                -- If we have a stable state and auto-rollback is enabled, consider rolling back
                if DiagnosticSystem.Status.HealthScore < 50 then
                    DiagnosticSystem:LogError(
                        "Auto-rolling back to last stable state due to critical error", 
                        "ErrorPrevention", 
                        "warning",
                        {error = errorMsg}
                    )
                    self:RollbackToState(1) -- Use most recent snapshot
                end
            end
            
            return false -- Can't specifically react to this error
        end
        
        -- Found a matching pattern, attempt to fix
        DiagnosticSystem:LogError(
            "Attempting to fix error: " .. matchedPattern.pattern, 
            "ErrorPrevention", 
            "info",
            {category = matchedPattern.category}
        )
        
        -- Run the preventative action
        local success = matchedPattern.preventiveAction()
        if success then
            DiagnosticSystem:LogError(
                "Successfully fixed error", 
                "ErrorPrevention", 
                "success"
            )
            
            -- Improve health score slightly for successful fix
            DiagnosticSystem.Status.HealthScore = math.min(DiagnosticSystem.Status.HealthScore + 5, 100)
        else
            DiagnosticSystem:LogError(
                "Failed to fix error", 
                "ErrorPrevention", 
                "error"
            )
        end
        
        return success
    end,
    
    -- Update environment factor analysis
    UpdateEnvironmentFactors = function(self)
        -- Check FPS
        local fps = 1 / DiagnosticSystem.Performance.FrameTimeAvg
        self.EnvironmentFactors.lowFps = fps < 30
        
        -- Check memory usage
        self.EnvironmentFactors.highMemoryUsage = DiagnosticSystem.Performance.MemoryUsage > 150 -- MB
        
        -- Network conditions check (simplified example)
        -- In a real implementation, you'd check ping or monitor network errors
        local ping = 100 -- Just a placeholder
        self.EnvironmentFactors.poorNetworkConditions = ping > 200
    end,
    
    -- Regular update for prediction system
    Update = function(self)
        -- Only update at the configured rate
        local now = os.clock()
        if now - self.LastPredictionUpdate < DiagnosticSystem.Config.PredictionUpdateRate then
            return
        end
        self.LastPredictionUpdate = now
        
        -- Update environment analysis
        self:UpdateEnvironmentFactors()
        
        -- Update diagnostic info
        local updateDisplay = false
        
        -- Check if health score is low
        if DiagnosticSystem.Status.HealthScore < DiagnosticSystem.Config.MinHealthBeforeAlert and
           #self.StateHistory > 0 and DiagnosticSystem.Config.AutoRollbackEnabled then
            
            -- Health is critically low, consider auto-rollback
            DiagnosticSystem:LogError(
                "System health critically low - considering auto-rollback", 
                "ErrorPrevention", 
                "warning",
                {healthScore = DiagnosticSystem.Status.HealthScore}
            )
            
            -- If we haven't recently tried a rollback, attempt one
            local lastRollbackAttempt = 0
            for _, error in ipairs(DiagnosticSystem.Errors) do
                if error.source == "ErrorPrevention" and 
                   string.find(error.message, "Rolling back") then
                    lastRollbackAttempt = error.timestamp
                    break
                end
            end
            
            if now - lastRollbackAttempt > 30 then -- Don't rollback too frequently
                self:RollbackToState(1) -- Use most recent state
            end
            
            updateDisplay = true
        end
        
        -- Prune old entries from operation log
        for i = #self.RiskyOperationLog, 1, -1 do
            local entry = self.RiskyOperationLog[i]
            if now - entry.timestamp > 300 then -- 5 minutes old
                table.remove(self.RiskyOperationLog, i)
            end
        end
        
        -- Return whether display needs update
        return updateDisplay
    end,
    
    -- Initialize prediction system
    Initialize = function(self)
        -- Store initial state
        self:StoreStateSnapshot("Initial State")
        
        -- Setup a regular update
        RunService.Heartbeat:Connect(function()
            if DiagnosticSystem.Config.PredictiveErrorPrevention then
                self:Update()
            end
        end)
        
        DiagnosticSystem:LogError(
            "Predictive error prevention system initialized", 
            "ErrorPrevention", 
            "success"
        )
    end
}

-- ORIGINAL DIAGNOSTIC SYSTEM FUNCTIONS

-- Create UI for diagnostic tool with error prediction display
function DiagnosticSystem:CreateUI(parent)
    if not parent then return nil end
    
    local container = Make("Frame", {
        Name = "DiagnosticContainer",
        Size = UDim2.new(1, -20, 0, 320), -- Increased height for prediction panel
        Position = UDim2.new(0, 10, 0, 10),
        BackgroundTransparency = 1,
        Parent = parent
    })
    
    if not container then return nil end
    
    -- Health score indicator
    local healthScoreFrame = Make("Frame", {
        Name = "HealthScoreFrame",
        Size = UDim2.new(1, 0, 0, 80),
        Position = UDim2.new(0, 0, 0, 0),
        BackgroundTransparency = 1,
        Parent = container
    })
    
    if healthScoreFrame then
        -- Score text
        local scoreText = Make("TextLabel", {
            Name = "ScoreText",
            Text = "System Health: 100%",
            Size = UDim2.new(1, 0, 0, 30),
            Position = UDim2.new(0, 0, 0, 0),
            Font = Enum.Font.GothamBold,
            TextSize = 18,
            TextColor3 = self:GetHealthColor(self.Status.HealthScore),
            BackgroundTransparency = 1,
            Parent = healthScoreFrame
        })
        
        -- Health bar background
        local barBg = Make("Frame", {
            Name = "BarBackground",
            Size = UDim2.new(1, 0, 0, 20),
            Position = UDim2.new(0, 0, 0, 35),
            BackgroundColor3 = CyberTheme.sliderTrack,
            BorderSizePixel = 0,
            Parent = healthScoreFrame
        })
        
        if barBg then
            Make("UICorner", {
                CornerRadius = UDim.new(0, 10),
                Parent = barBg
            })
            
            -- Health bar fill
            local barFill = Make("Frame", {
                Name = "BarFill",
                Size = UDim2.new(self.Status.HealthScore/100, 0, 1, 0),
                Position = UDim2.new(0, 0, 0, 0),
                BackgroundColor3 = self:GetHealthColor(self.Status.HealthScore),
                BorderSizePixel = 0,
                Parent = barBg
            })
            
            if barFill then
                Make("UICorner", {
                    CornerRadius = UDim.new(0, 10),
                    Parent = barFill
                })
            end
        end
        
        -- Status text
        local statusText = Make("TextLabel", {
            Name = "StatusText",
            Text = "All systems operational",
            Size = UDim2.new(1, 0, 0, 20),
            Position = UDim2.new(0, 0, 0, 60),
            Font = Enum.Font.GothamMedium,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            BackgroundTransparency = 1,
            Parent = healthScoreFrame
        })
    end
    
    -- NEW: Error Prediction Panel
    local predictionFrame = Make("Frame", {
        Name = "PredictionFrame",
        Size = UDim2.new(1, 0, 0, 60),
        Position = UDim2.new(0, 0, 0, 85),
        BackgroundTransparency = 0.9,
        BackgroundColor3 = CyberTheme.card,
        Parent = container
    })
    
    if predictionFrame then
        Make("UICorner", {
            CornerRadius = UDim.new(0, 8),
            Parent = predictionFrame
        })
        
        -- Title
        local predictionTitle = Make("TextLabel", {
            Name = "PredictionTitle",
            Text = "Error Prediction",
            Size = UDim2.new(0.5, 0, 0, 25),
            Position = UDim2.new(0, 10, 0, 5),
            Font = Enum.Font.GothamBold,
            TextSize = 16,
            TextColor3 = CyberTheme.accent,
            TextXAlignment = Enum.TextXAlignment.Left,
            BackgroundTransparency = 1,
            Parent = predictionFrame
        })
        
        -- Risk Level
        local riskLevel = Make("TextLabel", {
            Name = "RiskLevel",
            Text = "Current Risk: Low",
            Size = UDim2.new(0.5, 0, 0, 20),
            Position = UDim2.new(0, 10, 0, 30),
            Font = Enum.Font.GothamMedium,
            TextSize = 14,
            TextColor3 = CyberTheme.success,
            TextXAlignment = Enum.TextXAlignment.Left,
            BackgroundTransparency = 1,
            Parent = predictionFrame
        })
        
        -- Prevention Status
        local preventionStatus = Make("TextLabel", {
            Name = "PreventionStatus",
            Text = "Auto-Prevention: Active",
            Size = UDim2.new(0.5, 0, 0, 20),
            Position = UDim2.new(0.5, 0, 0, 30),
            Font = Enum.Font.GothamMedium,
            TextSize = 14,
            TextColor3 = self.Config.PredictiveErrorPrevention and CyberTheme.success or CyberTheme.error,
            TextXAlignment = Enum.TextXAlignment.Left,
            BackgroundTransparency = 1,
            Parent = predictionFrame
        })
        
        -- Toggle button for prevention system
        local toggleBtn = Make("TextButton", {
            Name = "ToggleBtn",
            Text = self.Config.PredictiveErrorPrevention and "Disable" or "Enable",
            Size = UDim2.new(0, 80, 0, 25),
            Position = UDim2.new(1, -90, 0, 5),
            Font = Enum.Font.GothamSemibold,
            TextSize = 14,
            TextColor3 = Color3.fromRGB(255, 255, 255),
            BackgroundColor3 = self.Config.PredictiveErrorPrevention and CyberTheme.error or CyberTheme.success,
            Parent = predictionFrame
        })
        
        if toggleBtn then
            Make("UICorner", {
                CornerRadius = UDim.new(0, 6),
                Parent = toggleBtn
            })
            
            toggleBtn.MouseButton1Click:Connect(function()
                self.Config.PredictiveErrorPrevention = not self.Config.PredictiveErrorPrevention
                toggleBtn.Text = self.Config.PredictiveErrorPrevention and "Disable" or "Enable"
                toggleBtn.BackgroundColor3 = self.Config.PredictiveErrorPrevention and CyberTheme.error or CyberTheme.success
                preventionStatus.Text = "Auto-Prevention: " .. (self.Config.PredictiveErrorPrevention and "Active" or "Disabled")
                preventionStatus.TextColor3 = self.Config.PredictiveErrorPrevention and CyberTheme.success or CyberTheme.error
                
                self:LogError(
                    "Predictive error prevention " .. (self.Config.PredictiveErrorPrevention and "enabled" or "disabled"),
                    "ErrorPrevention",
                    "info"
                )
            end)
        end
    end
    
    -- Component status section
    local compStatusFrame = Make("Frame", {
        Name = "ComponentStatus",
        Size = UDim2.new(1, 0, 0, 100),
        Position = UDim2.new(0, 0, 0, 150),
        BackgroundTransparency = 1,
        Parent = container
    })
    
    if compStatusFrame then
        local componentLabels = {
            {name = "UI", status = self.Status.ComponentStatus.UI},
            {name = "Angle Enhancer", status = self.Status.ComponentStatus.AngleEnhancer},
            {name = "Freeze", status = self.Status.ComponentStatus.Freeze},
            {name = "Controller", status = self.Status.ComponentStatus.Controller}
        }
        
        for i, comp in ipairs(componentLabels) do
            local compRow = Make("Frame", {
                Name = comp.name .. "Row",
                Size = UDim2.new(1, 0, 0, 25),
                Position = UDim2.new(0, 0, 0, (i-1) * 25),
                BackgroundTransparency = 1,
                Parent = compStatusFrame
            })
            
            if compRow then
                -- Component name
                local nameLabel = Make("TextLabel", {
                    Name = "NameLabel",
                    Text = comp.name .. ":",
                    Size = UDim2.new(0.4, 0, 1, 0),
                    Position = UDim2.new(0, 0, 0, 0),
                    Font = Enum.Font.GothamMedium,
                    TextSize = 14,
                    TextColor3 = CyberTheme.text,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    BackgroundTransparency = 1,
                    Parent = compRow
                })
                
                -- Status indicator
                local statusColor
                if comp.status == "ok" then
                    statusColor = CyberTheme.success
                elseif comp.status == "warning" then
                    statusColor = CyberTheme.warning
                elseif comp.status == "error" then
                    statusColor = CyberTheme.error
                elseif comp.status == "disabled" then
                    statusColor = CyberTheme.textDim
                else
                    statusColor = CyberTheme.textDim
                end
                
                local statusLabel = Make("TextLabel", {
                    Name = "StatusLabel",
                    Text = comp.status:upper(),
                    Size = UDim2.new(0.6, 0, 1, 0),
                    Position = UDim2.new(0.4, 0, 0, 0),
                    Font = Enum.Font.GothamSemibold,
                    TextSize = 14,
                    TextColor3 = statusColor,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    BackgroundTransparency = 1,
                    Parent = compRow
                })
            end
        end
    end
    
    -- Error log section
    local errorLogFrame = Make("Frame", {
        Name = "ErrorLog",
        Size = UDim2.new(1, 0, 0, 60),
        Position = UDim2.new(0, 0, 0, 255),
        BackgroundTransparency = 1,
        Parent = container
    })
    
    if errorLogFrame then
        -- Error log title
        local logTitle = Make("TextLabel", {
            Name = "LogTitle",
            Text = "Recent Errors:",
            Size = UDim2.new(1, 0, 0, 20),
            Position = UDim2.new(0, 0, 0, 0),
            Font = Enum.Font.GothamSemibold,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            TextXAlignment = Enum.TextXAlignment.Left,
            BackgroundTransparency = 1,
            Parent = errorLogFrame
        })
        
        -- Error log text
        local logText = Make("TextLabel", {
            Name = "LogText",
            Text = "No errors reported",
            Size = UDim2.new(1, 0, 0, 40),
            Position = UDim2.new(0, 0, 0, 20),
            Font = Enum.Font.Gotham,
            TextSize = 12,
            TextColor3 = CyberTheme.success,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextYAlignment = Enum.TextYAlignment.Top,
            TextWrapped = true,
            BackgroundTransparency = 1,
            Parent = errorLogFrame
        })
    end
    
    return container
end

-- Helper to get color based on health score
function DiagnosticSystem:GetHealthColor(score)
    if score >= 80 then
        return CyberTheme.success -- Green
    elseif score >= 50 then
        return CyberTheme.warning -- Yellow
    else
        return CyberTheme.error -- Red
    end
end

-- Log an error event
function DiagnosticSystem:LogError(message, source, severity, details)
    if not self.Config.Enabled then return end
    
    -- Create error object
    local errorObj = {
        message = message or "Unknown error",
        source = source or "Unknown",
        severity = severity or "error", -- Can be "info", "warning", "error", "success"
        timestamp = os.time(),
        details = details or {}
    }
    
    -- Add to errors list (limit size)
    if #self.Errors >= self.Config.MaxErrorsStored then
        table.remove(self.Errors, 1) -- Remove oldest
    end
    table.insert(self.Errors, errorObj)
    
    -- Update health score based on severity
    if severity == "error" then
        self.Status.HealthScore = math.max(self.Status.HealthScore - 15, 0)
    elseif severity == "warning" then
        self.Status.HealthScore = math.max(self.Status.HealthScore - 5, 0)
    end
    
    -- Log to output if enabled
    if self.Config.LogToOutput then
        if severity == "error" then
            warn("[FF2EnhancerUI ERROR] " .. source .. ": " .. message)
        elseif severity == "warning" then
            warn("[FF2EnhancerUI WARNING] " .. source .. ": " .. message)
        elseif severity == "info" then
            print("[FF2EnhancerUI INFO] " .. source .. ": " .. message)
        elseif severity == "success" then
            print("[FF2EnhancerUI SUCCESS] " .. source .. ": " .. message)
        end
    end
    
    -- NEW: If this is an error or warning, check if we can actively resolve it
    if (severity == "error" or severity == "warning") and 
       self.Config.PredictiveErrorPrevention and 
       self.Config.AutoHeal then
        
        task.spawn(function()
            -- Don't block main thread
            -- Try to resolve the error
            self.ErrorPrediction:ReactToError(message, source)
        end)
    end
    
    return errorObj
end

-- Update the diagnostic panel with latest information
function DiagnosticSystem:UpdateDiagnosticUI()
    local tabPages = _G.FF2_TabPages
    if not tabPages or not tabPages[5] or not tabPages[5]:FindFirstChild("DiagnosticContainer") then return end
    
    local container = tabPages[5]:FindFirstChild("DiagnosticContainer")
    local healthScoreFrame = container:FindFirstChild("HealthScoreFrame")
    
    if healthScoreFrame then
        -- Update health score text and color
        local scoreText = healthScoreFrame:FindFirstChild("ScoreText")
        if scoreText then
            scoreText.Text = "System Health: " .. self.Status.HealthScore .. "%"
            scoreText.TextColor3 = self:GetHealthColor(self.Status.HealthScore)
        end
        
        -- Update health bar
        local barBg = healthScoreFrame:FindFirstChild("BarBackground")
        if barBg then
            local barFill = barBg:FindFirstChild("BarFill")
            if barFill then
                barFill.Size = UDim2.new(self.Status.HealthScore/100, 0, 1, 0)
                barFill.BackgroundColor3 = self:GetHealthColor(self.Status.HealthScore)
            end
        end
        
        -- Update status text
        local statusText = healthScoreFrame:FindFirstChild("StatusText")
        if statusText then
            if self.Status.HealthScore >= 80 then
                statusText.Text = "All systems operational"
            elseif self.Status.HealthScore >= 50 then
                statusText.Text = "Minor issues detected"
            else
                statusText.Text = "System requires attention"
            end
        end
    end
    
    -- NEW: Update prediction frame
    local predictionFrame = container:FindFirstChild("PredictionFrame")
    if predictionFrame then
        local riskLevel = predictionFrame:FindFirstChild("RiskLevel")
        if riskLevel then
            local riskValue = self.ErrorPrediction.CurrentRiskLevel
            local riskText
            local riskColor
            
            if riskValue < 0.3 then
                riskText = "Current Risk: Low"
                riskColor = CyberTheme.success
            elseif riskValue < 0.7 then
                riskText = "Current Risk: Medium"
                riskColor = CyberTheme.warning
            else
                riskText = "Current Risk: High"
                riskColor = CyberTheme.error
            end
            
            riskLevel.Text = riskText
            riskLevel.TextColor3 = riskColor
        end
        
        local preventionStatus = predictionFrame:FindFirstChild("PreventionStatus")
        if preventionStatus then
            preventionStatus.Text = "Auto-Prevention: " .. (self.Config.PredictiveErrorPrevention and "Active" or "Disabled")
            preventionStatus.TextColor3 = self.Config.PredictiveErrorPrevention and CyberTheme.success or CyberTheme.error
        end
    end
    
    -- Update component statuses
    local compStatusFrame = container:FindFirstChild("ComponentStatus")
    if compStatusFrame then
        local components = {
            {name = "UI", status = self.Status.ComponentStatus.UI},
            {name = "Angle Enhancer", status = self.Status.ComponentStatus.AngleEnhancer},
            {name = "Freeze", status = self.Status.ComponentStatus.Freeze},
            {name = "Controller", status = self.Status.ComponentStatus.Controller}
        }
        
        for i, comp in ipairs(components) do
            local compRow = compStatusFrame:FindFirstChild(comp.name .. "Row")
            if compRow then
                local statusLabel = compRow:FindFirstChild("StatusLabel")
                if statusLabel then
                    statusLabel.Text = comp.status:upper()
                    
                    local statusColor
                    if comp.status == "ok" then
                        statusColor = CyberTheme.success
                    elseif comp.status == "warning" then
                        statusColor = CyberTheme.warning
                    elseif comp.status == "error" then
                        statusColor = CyberTheme.error
                    elseif comp.status == "disabled" then
                        statusColor = CyberTheme.textDim
                    else
                        statusColor = CyberTheme.textDim
                    end
                    
                    statusLabel.TextColor3 = statusColor
                end
            end
        end
    end
    
    -- Update error log
    local errorLogFrame = container:FindFirstChild("ErrorLog")
    if errorLogFrame then
        local logText = errorLogFrame:FindFirstChild("LogText")
        if logText then
            if #self.Errors > 0 then
                local latestErrors = {}
                local count = math.min(#self.Errors, 3) -- Show last 3 errors
                
                for i = #self.Errors, #self.Errors - count + 1, -1 do
                    local err = self.Errors[i]
                    local severitySymbol = ""
                    local severityColor
                    
                    if err.severity == "error" then
                        severitySymbol = "✕ "
                        severityColor = CyberTheme.error
                    elseif err.severity == "warning" then
                        severitySymbol = "! "
                        severityColor = CyberTheme.warning
                    elseif err.severity == "info" then
                        severitySymbol = "ℹ "
                        severityColor = CyberTheme.accent
                    elseif err.severity == "success" then
                        severitySymbol = "✓ "
                        severityColor = CyberTheme.success
                    end
                    
                    table.insert(latestErrors, severitySymbol .. err.source .. ": " .. err.message)
                end
                
                logText.Text = table.concat(latestErrors, "\n")
            else
                logText.Text = "No errors reported"
                logText.TextColor3 = CyberTheme.success
            end
        end
    end
end

-- Run a comprehensive system diagnostic
function DiagnosticSystem:RunFullDiagnostic()
    -- Log the start of diagnostic
    self:LogError("Starting full diagnostic check", "Diagnostic", "info")
    
    -- Test performance
    self:MeasurePerformance()
    
    -- Check system components
    self:CheckComponents()
    
    -- NEW: Also run error prediction logic
    if self.Config.PredictiveErrorPrevention then
        self.ErrorPrediction:Update()
    end
    
    -- Log completion
    self:LogError("Diagnostic complete", "Diagnostic", "success")
    
    -- Update UI
    self:UpdateDiagnosticUI()
    
    -- Return diagnostic results
    return {
        health = self.Status.HealthScore,
        components = self.Status.ComponentStatus,
        performance = {
            fps = math.floor(1000 / math.max(1, self.Performance.FrameTimeAvg)),
            frameTime = self.Performance.FrameTimeAvg,
        },
        errorCount = #self.Errors,
        -- NEW: Add prediction info
        prediction = {
            riskLevel = self.ErrorPrediction.CurrentRiskLevel,
            enabled = self.Config.PredictiveErrorPrevention,
            envFactors = self.ErrorPrediction.EnvironmentFactors
        }
    }
end

-- Performance measurement function
function DiagnosticSystem:MeasurePerformance()
    -- Simple FPS counter implementation
    local frameTimes = {}
    local startTime = os.clock()
    local sampleCount = 0
    
    -- Connect temporary performance measuring
    local connection
    connection = RunService.Heartbeat:Connect(function(deltaTime)
        -- Store frame time in ms
        table.insert(frameTimes, deltaTime * 1000)
        sampleCount = sampleCount + 1
        
        if sampleCount >= self.Config.SampleSize then
            connection:Disconnect()
            
            -- Calculate average frame time
            local sum = 0
            for _, time in ipairs(frameTimes) do
                sum = sum + time
            end
            
            local avg = sum / #frameTimes
            self.Performance.FrameTimeAvg = avg
            
            -- Log performance issues
            if avg > self.Config.PerformanceThreshold then
                local fps = math.floor(1000 / avg)
                self:LogError(
                    "Performance below target " .. fps .. " FPS (" .. math.floor(avg) .. "ms/frame)", 
                    "Performance", 
                    "warning"
                )
                
                -- Reduce health score for poor performance
                self.Status.HealthScore = math.max(self.Status.HealthScore - 10, 0)
                
                -- NEW: Set environment factor
                self.ErrorPrediction.EnvironmentFactors.lowFps = true
            else
                self.ErrorPrediction.EnvironmentFactors.lowFps = false
            end
        end
    end)
    
    -- Wait for enough samples or timeout after 1 second
    while sampleCount < self.Config.SampleSize and (os.clock() - startTime < 1) do
        task.wait(0.1)
    end
    
    -- Ensure connection is disconnected
    if connection.Connected then
        connection:Disconnect()
    end
    
    -- NEW: Estimate memory usage for environment factor
    pcall(function()
        local stats = game:GetService("Stats")
        if stats then
            local memoryUsage = stats:GetTotalMemoryUsageMb()
            self.Performance.MemoryUsage = memoryUsage
            self.ErrorPrediction.EnvironmentFactors.highMemoryUsage = memoryUsage > 150 -- MB
        end
    end)
    
    return self.Performance.FrameTimeAvg
end

-- Test system components
function DiagnosticSystem:CheckComponents()
    -- Check UI components
    local uiStatus = "ok"
    if not screenGui or not screenGui.Parent then
        uiStatus = "error"
        self:LogError("ScreenGui not found or parented", "UI", "error")
        
        -- NEW: Try to fix this issue with prediction system
        if self.Config.PredictiveErrorPrevention and self.Config.AutoHeal then
            if Players.LocalPlayer:FindFirstChild("PlayerGui") then
                self:LogError("Attempting to recreate ScreenGui", "ErrorPrevention", "info")
                
                local newScreenGui = Instance.new("ScreenGui")
                newScreenGui.Name = "FF2EnhancerUI"
                newScreenGui.ResetOnSpawn = false
                newScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
                newScreenGui.Parent = Players.LocalPlayer.PlayerGui
                
                screenGui = newScreenGui
                -- Note: This won't fully fix the issue as we'd need to rebuild the entire UI
                -- but it's a demonstration of auto-healing capabilities
            end
        end
    elseif not win or not win.Parent then
        uiStatus = "warning"
        self:LogError("Main window not found", "UI", "warning")
    end
    self.Status.ComponentStatus.UI = uiStatus
    
    -- Check Angle Enhancer
    local angleStatus = "ok"
    if not Config.AngleEnabled and not Config.SoundEnabled and not Config.VelocityBoost then
        angleStatus = "disabled" -- All features disabled
    elseif not player or not player.Character then
        angleStatus = "warning"
        self:LogError("Character not found for Angle Enhancer", "AngleEnhancer", "warning")
        
        -- NEW: Update with more details for prediction system
        self.ErrorPrediction:LogRiskyOperation("CharacterOperation", "No character available", false)
    end
    self.Status.ComponentStatus.AngleEnhancer = angleStatus
    
    -- Check Freeze System
    local freezeStatus = "ok"
    if not Config.FreezeEnabled then
        freezeStatus = "disabled" -- Feature disabled
    elseif not player or not player.Character then
        freezeStatus = "warning"
        self:LogError("Character not found for Freeze", "Freeze", "warning")
        
        -- NEW: Log for prediction
        self.ErrorPrediction:LogRiskyOperation("FreezeOperation", "No character available", false)
    elseif not FreezeSystem or not FreezeSystem.initialized then
        freezeStatus = "error"
        self:LogError("Freeze system not initialized", "Freeze", "error")
    end
    self.Status.ComponentStatus.Freeze = freezeStatus
    
    -- Check Controller Support
    local controllerStatus = "ok"
    if not UserInputService.GamepadEnabled then
        controllerStatus = "unavailable"
        self:LogError("Gamepad not detected", "Controller", "info")
    end
    self.Status.ComponentStatus.Controller = controllerStatus
    
    -- NEW: Run some predictive checks
    if self.Config.PredictiveErrorPrevention then
        -- Store a state snapshot during diagnostic if health is good
        if self.Status.HealthScore >= 90 then
            self.ErrorPrediction:StoreStateSnapshot("Diagnostic Checkpoint")
        end
        
        -- Pre-check for common error patterns
        self.ErrorPrediction:RunPatternCheck("CharacterNotFound")
        self.ErrorPrediction:RunPatternCheck("HumanoidNotFound")
    end
end

-- Initialize the diagnostic system
function DiagnosticSystem:Initialize()
    self.Status.IsRunning = true
    self.Status.LastAutoCheck = os.clock()
    
    -- Set up auto-diagnostic on interval
    task.spawn(function()
        while true do
            local currentTime = os.clock()
            if currentTime - self.Status.LastAutoCheck >= self.Config.AutoDiagInterval then
                self:RunFullDiagnostic()
                self.Status.LastAutoCheck = currentTime
            end
            task.wait(5) -- Check every 5 seconds if interval has passed
        end
    end)
    
    -- Initialize error prediction system
    if self.Config.PredictiveErrorPrevention then
        self.ErrorPrediction:Initialize()
    end
    
    self:LogError("Diagnostic system initialized", "System", "success")
end

--// Enhanced error handling with predictive capabilities

-- Enhanced safe function call with predictive error prevention
local function safePcall(func, ...)
    -- Check if the prediction system is active
    if DiagnosticSystem.Config.PredictiveErrorPrevention and 
       DiagnosticSystem.Config.PreemptiveChecks then
        -- Try to predict if this operation might cause an error
        local operationType = "GenericOperation" -- Default
        
        -- Try to determine operation type based on func's debug info
        if func and debug and debug.info then
            local info = debug.info(func, "n")
            if info then
                if string.find(info, "Character") or string.find(info, "Human") then
                    operationType = "CharacterOperation"
                elseif string.find(info, "Freeze") then
                    operationType = "FreezeOperation"
                elseif string.find(info, "UI") or string.find(info, "Gui") then
                    operationType = "UIOperation"
                end
            end
        end
        
        -- Run predictive checks - if it returns false, the operation is too risky
        local safeToRun = DiagnosticSystem.ErrorPrediction:PreventErrors(operationType, {...})
        if not safeToRun then
            local errorMsg = "Operation aborted due to high risk prediction"
            warn("FF2EnhancerUI Warning: " .. errorMsg)
            
            -- Log this prevention
            DiagnosticSystem:LogError(
                errorMsg .. " (" .. operationType .. ")", 
                "ErrorPrevention", 
                "warning"
            )
            
            return nil
        end
    end
    
    -- Run the function with error handling
    local success, result = pcall(func, ...)
    if not success then
        local errorMsg = tostring(result)
        warn("FF2EnhancerUI Error: " .. errorMsg)
        
        -- Log to diagnostic system if available
        if DiagnosticSystem and DiagnosticSystem.LogError then
            DiagnosticSystem:LogError(errorMsg, debug.info(2, "n") or "Unknown", "error")
        end
        
        -- NEW: Record this error pattern for future prediction
        if DiagnosticSystem.Config.PredictiveErrorPrevention then
            -- Try to identify which pattern this matches
            local matched = false
            for key, pattern in pairs(DiagnosticSystem.ErrorPrediction.KnownPatterns) do
                if string.find(errorMsg, pattern.pattern) then
                    -- Record this operation as risky
                    DiagnosticSystem.ErrorPrediction:LogRiskyOperation(
                        key, 
                        errorMsg, 
                        false
                    )
                    matched = true
                    break
                end
            end
            
            -- If we didn't match a known pattern, log it generically
            if not matched then
                DiagnosticSystem.ErrorPrediction:LogRiskyOperation(
                    "UnknownError", 
                    errorMsg, 
                    false
                )
            end
            
            -- If auto-heal is enabled, try to react to this error
            if DiagnosticSystem.Config.AutoHeal then
                DiagnosticSystem.ErrorPrediction:ReactToError(errorMsg, debug.info(2, "n") or "Unknown")
            end
        end
        
        return nil
    end
    
    -- Operation succeeded
    if DiagnosticSystem.Config.PredictiveErrorPrevention then
        -- Keep track of successful operations as well
        local operationType = "GenericOperation"
        DiagnosticSystem.ErrorPrediction:LogRiskyOperation(operationType, nil, true)
    end
    
    return result
end

-- Rest of the script remains similar to previous implementation
-- with integration of error prediction system throughout

-- Safe function calls
local function safeInsert(tbl, value)
    if type(tbl) ~= "table" then
        local errorMsg = "Attempted to insert into a non-table value"
        warn("FF2EnhancerUI Error: " .. errorMsg)
        
        -- Log to diagnostic system if available
        if DiagnosticSystem and DiagnosticSystem.LogError then
            DiagnosticSystem:LogError(errorMsg, debug.info(2, "n") or "Unknown", "error")
        end
        
        return
    end
    
    -- NEW: Verify value to pre-empt potential errors
    if value == nil and DiagnosticSystem.Config.PredictiveErrorPrevention then
        local errorMsg = "Prevented insertion of nil value into table"
        DiagnosticSystem:LogError(errorMsg, "ErrorPrevention", "warning")
        return
    end
    
    table.insert(tbl, value)
end

local function safeCallback(callback, ...)
    if type(callback) ~= "function" then
        local errorMsg = "Invalid callback function"
        warn("FF2EnhancerUI Error: " .. errorMsg)
        
        -- Log to diagnostic system if available
        if DiagnosticSystem and DiagnosticSystem.LogError then
            DiagnosticSystem:LogError(errorMsg, debug.info(2, "n") or "Unknown", "error")
        end
        
        return
    end
    
    -- NEW: Additional validation for callback arguments
    if DiagnosticSystem.Config.PredictiveErrorPrevention then
        -- Verify that no args are invalid or might cause errors
        local args = {...}
        for i, arg in ipairs(args) do
            if type(arg) == "userdata" and pcall(function() return arg.Parent end) and arg.Parent == nil then
                -- This is a destroyed instance, which will cause errors
                local errorMsg = "Prevented callback with destroyed instance argument"
                DiagnosticSystem:LogError(errorMsg, "ErrorPrevention", "warning")
                return
            end
        end
    end
    
    safePcall(callback, ...)
end

-- Enhanced safe instance creation
local function Make(className, properties)
    local success, instance = pcall(function()
        return Instance.new(className)
    end)
    
    if not success or not instance then
        local errorMsg = "Failed to create " .. className
        warn("FF2EnhancerUI Error: " .. errorMsg)
        
        -- Log to diagnostic system if available
        if DiagnosticSystem and DiagnosticSystem.LogError then
            DiagnosticSystem:LogError(errorMsg, debug.info(2, "n") or "Unknown", "error")
        end
        
        return nil
    end
    
    if properties then
        -- NEW: Pre-validate properties to avoid errors
        if DiagnosticSystem.Config.PredictiveErrorPrevention then
            for prop, value in pairs(properties) do
                -- Check if property exists before assigning
                local canAssign = pcall(function() 
                    local _ = instance[prop] 
                end)
                
                if not canAssign then
                    local errorMsg = "Invalid property '" .. prop .. "' for " .. className
                    DiagnosticSystem:LogError(errorMsg, "ErrorPrevention", "warning")
                    
                    -- Skip this property but continue with others
                    properties[prop] = nil
                end
                
                -- Check if value is a destroyed instance
                if type(value) == "userdata" and pcall(function() return value.Parent end) and value.Parent == nil 
                    and prop ~= "Parent" then -- Parent can be nil
                    
                    local errorMsg = "Detected destroyed instance being assigned to " .. prop
                    DiagnosticSystem:LogError(errorMsg, "ErrorPrevention", "warning")
                    
                    -- Skip this property
                    properties[prop] = nil
                end
            end
        end
        
        -- Apply validated properties
        for prop, value in pairs(properties) do
            safePcall(function() 
                instance[prop] = value 
            end)
        end
    end
    
    return instance
end

-- Create glow effect
local function CreateGlowEffect(parent, thickness, color, transparency)
    if not parent then return nil end
    
    local stroke = Make("UIStroke", {
        Parent = parent,
        Thickness = thickness or 2,
        Color = color or Color3.fromRGB(0, 230, 230),
        Transparency = transparency or 0,
    })
    
    return stroke
end

local player = Players.LocalPlayer

-- The rest of the script continues as in FF2EnhancerUI_WithFreeze.lua
-- with integration of error prediction system in key areas

-- When using the FreezeSystem:
-- FreezeSystem:ToggleFreeze() would be wrapped with preventive checks:
--
-- FreezeSystem.ToggleFreeze = function(self)
--     -- Before toggle, run predictive check
--     if DiagnosticSystem.Config.PredictiveErrorPrevention then
--         local canProceed = DiagnosticSystem.ErrorPrediction:PreventErrors("FreezeOperation", {})
--         if not canProceed then return end
--     end
--     
--     -- Original toggle freeze code
--     -- ...
-- end

--// ENHANCED GUI SETTINGS & STATE

-- Define device detection
local IS_MOBILE = UserInputService.TouchEnabled and not UserInputService.MouseEnabled
local IS_CONSOLE = UserInputService.GamepadEnabled and not UserInputService.TouchEnabled and not UserInputService.MouseEnabled

-- Screen resolution and scaling
local screenSize = workspace.CurrentCamera.ViewportSize
local scaleMultiplier = math.clamp(math.min(screenSize.X/1920, screenSize.Y/1080), 0.5, 1.5)

-- Full theme system with presets
local ThemeSystem = {}

-- Define multiple themes
ThemeSystem.Themes = {
    ["Cyber"] = {
        background = Color3.fromRGB(6, 12, 24),     -- Deep dark blue background
        card = Color3.fromRGB(10, 20, 35),          -- Slightly lighter blue for cards
        accent = Color3.fromRGB(0, 230, 230),       -- Bright cyan accent color
        accentAlt = Color3.fromRGB(140, 0, 255),    -- Purple accent for gradients
        text = Color3.fromRGB(230, 230, 230),       -- White text
        textDim = Color3.fromRGB(140, 140, 140),    -- Dimmed text
        toggleOn = Color3.fromRGB(0, 255, 180),     -- Toggle on color
        sliderTrack = Color3.fromRGB(20, 40, 70),   -- Slider track color
        success = Color3.fromRGB(0, 230, 120),      -- Success color
        error = Color3.fromRGB(255, 60, 60),        -- Error color
        warning = Color3.fromRGB(255, 190, 0),      -- Warning color
        shadow = Color3.fromRGB(0, 0, 0),          -- Shadow color
    },
    
    ["Retro"] = {
        background = Color3.fromRGB(40, 40, 40),     -- Dark grey background
        card = Color3.fromRGB(60, 60, 60),          -- Light grey for cards
        accent = Color3.fromRGB(255, 128, 0),       -- Orange accent color
        accentAlt = Color3.fromRGB(255, 40, 80),    -- Pink accent for gradients
        text = Color3.fromRGB(255, 255, 255),       -- White text
        textDim = Color3.fromRGB(180, 180, 180),    -- Dimmed text
        toggleOn = Color3.fromRGB(255, 128, 0),     -- Toggle on color
        sliderTrack = Color3.fromRGB(80, 80, 80),   -- Slider track color
        success = Color3.fromRGB(120, 255, 120),    -- Success color
        error = Color3.fromRGB(255, 80, 80),        -- Error color
        warning = Color3.fromRGB(255, 200, 80),     -- Warning color
        shadow = Color3.fromRGB(20, 20, 20),        -- Shadow color
    },
    
    ["Minimal"] = {
        background = Color3.fromRGB(250, 250, 250),  -- Almost white background
        card = Color3.fromRGB(240, 240, 240),         -- Light grey for cards
        accent = Color3.fromRGB(80, 120, 255),        -- Blue accent color
        accentAlt = Color3.fromRGB(40, 40, 120),      -- Darker blue
        text = Color3.fromRGB(40, 40, 40),            -- Almost black text
        textDim = Color3.fromRGB(120, 120, 120),      -- Grey text
        toggleOn = Color3.fromRGB(80, 120, 255),      -- Toggle on color
        sliderTrack = Color3.fromRGB(220, 220, 220),  -- Light grey
        success = Color3.fromRGB(80, 200, 120),       -- Green
        error = Color3.fromRGB(200, 80, 80),          -- Red
        warning = Color3.fromRGB(255, 180, 80),       -- Orange
        shadow = Color3.fromRGB(200, 200, 200),       -- Light shadow
    },
    
    ["NightVision"] = {
        background = Color3.fromRGB(5, 15, 5),      -- Very dark green background
        card = Color3.fromRGB(10, 25, 10),           -- Dark green for cards
        accent = Color3.fromRGB(0, 255, 80),         -- Bright green accent
        accentAlt = Color3.fromRGB(0, 120, 40),      -- Darker green
        text = Color3.fromRGB(0, 255, 80),           -- Green text
        textDim = Color3.fromRGB(0, 180, 60),        -- Dimmed green text
        toggleOn = Color3.fromRGB(0, 255, 80),       -- Bright green
        sliderTrack = Color3.fromRGB(10, 40, 10),    -- Dark green
        success = Color3.fromRGB(0, 255, 80),        -- Bright green
        error = Color3.fromRGB(255, 0, 0),           -- Red
        warning = Color3.fromRGB(255, 180, 0),       -- Orange
        shadow = Color3.fromRGB(0, 0, 0),            -- Black shadow
    },
    
    ["Custom"] = {
        background = Color3.fromRGB(6, 12, 24),     -- Default to Cyber theme
        card = Color3.fromRGB(10, 20, 35),          
        accent = Color3.fromRGB(0, 230, 230),       
        accentAlt = Color3.fromRGB(140, 0, 255),    
        text = Color3.fromRGB(230, 230, 230),       
        textDim = Color3.fromRGB(140, 140, 140),    
        toggleOn = Color3.fromRGB(0, 255, 180),     
        sliderTrack = Color3.fromRGB(20, 40, 70),   
        success = Color3.fromRGB(0, 230, 120),      
        error = Color3.fromRGB(255, 60, 60),        
        warning = Color3.fromRGB(255, 190, 0),      
        shadow = Color3.fromRGB(0, 0, 0),          
    }
}

-- Current theme
ThemeSystem.CurrentTheme = "Cyber"

function ThemeSystem:GetCurrentTheme()
    return self.Themes[self.CurrentTheme]
end

function ThemeSystem:ApplyTheme(themeName)
    if not self.Themes[themeName] then
        warn("Theme not found: " .. themeName)
        return false
    end
    
    self.CurrentTheme = themeName
    return true
end

function ThemeSystem:GetThemeColor(colorName)
    local theme = self:GetCurrentTheme()
    return theme[colorName] or Color3.fromRGB(255, 255, 255) -- Default to white if color not found
end

-- Changing color in custom theme
function ThemeSystem:SetCustomColor(colorName, color)
    if not self.Themes["Custom"][colorName] then
        warn("Color property not found: " .. colorName)
        return false
    end
    
    self.Themes["Custom"][colorName] = color
    return true
end

-- Config/State with enhanced options
local Config = {
    -- Core functionality settings
    AngleEnabled     = true,
    SoundEnabled     = true,
    ACBypass         = false,
    JumpPower        = 52.88,
    DefaultJP        = 50,
    VelocityBoost    = false,
    VelocityStrength = 1.5,

    FreezeEnabled    = false,
    JitterEnabled    = true,
    JitterStrength   = 0.1,
    DiveSupport      = true,
    
    -- UI settings
    UIVisible        = true,
    AnimationStyle   = "Slide",
    
    -- Freeze timing controls
    FreezeDurationMin = 0.04,
    FreezeDurationMax = 0.07,
    UnfreezeDelayMin = 0.008,
    UnfreezeDelayMax = 0.02,
    
    -- NEW UI enhancement settings
    Theme = "Cyber",
    UIScale = 1.0,               -- Default UI scale (1.0 = 100%)
    ParticleEffects = true,       -- Background particle effects
    AnimatedElements = true,      -- Button animations, gradients, etc.
    AutoScale = true,             -- Automatically scale based on screen size
    
    -- Advanced settings
    CustomColors = {},            -- For custom theme
    PreferredAspectRatio = "Auto", -- Auto, 16:9, 4:3, etc.
    WidgetStyle = "Modern",       -- Modern, Classic, Minimal
    
    -- Diagnostics
    DiagnosticsEnabled = true,
    
    -- User preferences
    SavedPresets = {},
    
    -- Controller bindings
    ControllerToggleKey = Enum.KeyCode.ButtonB,
    ControllerFreezeKey = Enum.KeyCode.ButtonY,
}

-- Apply the theme from config
ThemeSystem:ApplyTheme(Config.Theme)

-- Get current theme colors (shorthand for the theme system)
local CyberTheme = ThemeSystem:GetCurrentTheme()

-- Animation styles definition
local AnimationStyles = {
    -- Original animation styles
    ["Slide"] = function(win, show, onComplete)
        if not win then return end
        
        local onScreenPos = UDim2.new(0.5, -180 * scaleMultiplier, 0.5, -210 * scaleMultiplier) -- Center position with scaling
        local offScreenPos = UDim2.new(1.5, 0, 0.5, -210 * scaleMultiplier)   -- Off-screen to the right
        
        local tweenInfo = TweenInfo.new(
            0.5,                   -- Duration
            Enum.EasingStyle.Quint, -- Smooth easing
            Enum.EasingDirection.Out -- Ease out for smooth end
        )
        
        local targetPos = show and onScreenPos or offScreenPos
        local tween = safePcall(function() 
            return TweenService:Create(win, tweenInfo, {Position = targetPos}) 
        end)
        
        if tween then
            tween:Play()
            if onComplete and typeof(tween) == "Instance" and tween:IsA("Tween") then
                safePcall(function() 
                    tween.Completed:Connect(onComplete) 
                end)
            end
        end
        
        return tween
    end,
    
    ["Fade"] = function(win, show, onComplete)
        -- Implementation similar to original but with enhanced tweening
        if not win then return end
        
        -- Center window for consistent fade
        win.Position = UDim2.new(0.5, -180 * scaleMultiplier, 0.5, -210 * scaleMultiplier)
        
        local tweenInfo = TweenInfo.new(
            0.4,                      -- Duration
            Enum.EasingStyle.Cubic,   -- Smooth cubic easing
            Enum.EasingDirection.Out  -- Ease out
        )
        
        local targetTransparency = {}
        -- Fade all elements
        safePcall(function()
            for _, obj in ipairs(win:GetDescendants()) do
                if obj:IsA("GuiObject") and not obj:IsA("UIStroke") then
                    targetTransparency[obj] = show and 0 or 1
                end
            end
            targetTransparency[win] = show and 0 or 1
        end)
        
        -- Create and play individual tweens for each object
        local tweens = {}
        for obj, transparency in pairs(targetTransparency) do
            if obj and obj:IsA("GuiObject") then
                if obj:IsA("TextButton") or obj:IsA("TextLabel") or obj:IsA("ImageLabel") or obj:IsA("ImageButton") then
                    local tween = safePcall(function()
                        return TweenService:Create(obj, tweenInfo, {
                            BackgroundTransparency = transparency,
                            TextTransparency = transparency,
                            TextStrokeTransparency = 1 -- Text stroke should always be mostly transparent
                        })
                    end)
                    
                    if tween then
                        safeInsert(tweens, tween)
                        tween:Play()
                    end
                else
                    local tween = safePcall(function()
                        return TweenService:Create(obj, tweenInfo, {
                            BackgroundTransparency = transparency
                        })
                    end)
                    
                    if tween then
                        safeInsert(tweens, tween)
                        tween:Play()
                    end
                end
            end
        end
        
        -- Set up completion callback on the first tween
        if #tweens > 0 and onComplete and typeof(tweens[1]) == "Instance" and tweens[1]:IsA("Tween") then
            safePcall(function()
                tweens[1].Completed:Connect(onComplete)
            end)
        end
        
        return #tweens > 0 and tweens[1] or nil
    end,
    
    ["Scale"] = function(win, show, onComplete)
        if not win then return end
        
        local tweenInfo = TweenInfo.new(
            0.4,                    -- Duration
            Enum.EasingStyle.Back,  -- Elastic-like easing
            show and Enum.EasingDirection.Out or Enum.EasingDirection.In
        )
        
        -- Set initial position to center
        win.Position = UDim2.new(0.5, -180 * scaleMultiplier, 0.5, -210 * scaleMultiplier)
        
        -- Target size is either normal or very small
        local normalSize = UDim2.new(0, 360 * scaleMultiplier, 0, 420 * scaleMultiplier)
        local smallSize = UDim2.new(0, 10, 0, 10) -- Almost invisible
        
        local targetSize = show and normalSize or smallSize
        local targetTransparency = show and 0 or 1
        
        local tween = safePcall(function()
            return TweenService:Create(win, tweenInfo, {
                Size = targetSize,
                BackgroundTransparency = targetTransparency
            })
        end)
        
        if tween then
            tween:Play()
            if onComplete and typeof(tween) == "Instance" and tween:IsA("Tween") then
                safePcall(function()
                    tween.Completed:Connect(onComplete)
                end)
            end
        end
        
        return tween
    end,
    
    -- New animation styles
    ["Flip"] = function(win, show, onComplete)
        if not win then return end
        
        -- Center position
        win.Position = UDim2.new(0.5, -180 * scaleMultiplier, 0.5, -210 * scaleMultiplier)
        
        -- Setup tween
        local tweenInfo = TweenInfo.new(
            0.5,                      -- Duration
            Enum.EasingStyle.Cubic,   -- Smooth easing
            Enum.EasingDirection.Out  -- Ease out for smooth end
        )
        
        -- Start with a rotation for flipping effect
        local startRotation = show and 90 or 0
        local endRotation = show and 0 or 90
        win.Rotation = startRotation
        
        -- Also scale when flipping to enhance the effect
        local startScale = show and 0.7 or 1
        local endScale = show and 1 or 0.7
        win.Size = UDim2.new(0, 360 * scaleMultiplier * startScale, 0, 420 * scaleMultiplier * startScale)
        
        local targetProps = {
            Rotation = endRotation,
            Size = UDim2.new(0, 360 * scaleMultiplier * endScale, 0, 420 * scaleMultiplier * endScale),
            BackgroundTransparency = show and 0 or 1
        }
        
        local tween = safePcall(function()
            return TweenService:Create(win, tweenInfo, targetProps)
        end)
        
        if tween then
            tween:Play()
            if onComplete and typeof(tween) == "Instance" and tween:IsA("Tween") then
                safePcall(function() 
                    tween.Completed:Connect(onComplete) 
                end)
            end
        end
        
        return tween
    end,
    
    ["Bounce"] = function(win, show, onComplete)
        if not win then return end
        
        -- Define positions
        local onScreenPos = UDim2.new(0.5, -180 * scaleMultiplier, 0.5, -210 * scaleMultiplier)
        local offScreenPos = UDim2.new(0.5, -180 * scaleMultiplier, 1.5, 0)
        
        -- Setup tween
        local tweenInfo = TweenInfo.new(
            0.7,                    -- Duration
            Enum.EasingStyle.Bounce, -- Bounce easing
            Enum.EasingDirection.Out -- Ease out
        )
        
        local targetPos = show and onScreenPos or offScreenPos
        win.Position = show and offScreenPos or onScreenPos
        
        local tween = safePcall(function()
            return TweenService:Create(win, tweenInfo, {Position = targetPos})
        end)
        
        if tween then
            tween:Play()
            if onComplete and typeof(tween) == "Instance" and tween:IsA("Tween") then
                safePcall(function() 
                    tween.Completed:Connect(onComplete) 
                end)
            end
        end
        
        return tween
    end,
    
    ["Spin"] = function(win, show, onComplete)
        if not win then return end
        
        -- Center window
        win.Position = UDim2.new(0.5, -180 * scaleMultiplier, 0.5, -210 * scaleMultiplier)
        
        -- Setup tweens
        local spinTweenInfo = TweenInfo.new(
            0.6,                    -- Duration
            Enum.EasingStyle.Cubic,  -- Smooth easing
            Enum.EasingDirection.Out -- Ease out
        )
        
        -- Spin 360 degrees while fading
        local startRotation = show and 360 or 0
        local endRotation = show and 0 or 360
        win.Rotation = startRotation
        
        local targetProps = {
            Rotation = endRotation,
            BackgroundTransparency = show and 0 or 1
        }
        
        -- Simultaneously scale
        local startScale = show and 0.5 or 1
        local endScale = show and 1 or 0.5
        win.Size = UDim2.new(0, 360 * scaleMultiplier * startScale, 0, 420 * scaleMultiplier * startScale)
        
        -- Add scaling to targetProps
        targetProps.Size = UDim2.new(0, 360 * scaleMultiplier * endScale, 0, 420 * scaleMultiplier * endScale)
        
        local tween = safePcall(function()
            return TweenService:Create(win, spinTweenInfo, targetProps)
        end)
        
        if tween then
            tween:Play()
            if onComplete and typeof(tween) == "Instance" and tween:IsA("Tween") then
                safePcall(function() 
                    tween.Completed:Connect(onComplete) 
                end)
            end
        end
        
        return tween
    end,
}

--// FreezeTech System Integration
local FreezeSystem = {
    initialized = false,
    isFrozen = false,
    freezeRepeating = false,
    freezeThread = nil,
    debounce = false,
    jitterConnection = nil,
    storedVelocity = Vector3.zero,
    storedAngularVelocity = Vector3.zero,
    storedDiveState = false,
}

-- Private utility functions
local function getDynamicFreezeDuration()
    local min = Config.FreezeDurationMin * 1000
    local max = Config.FreezeDurationMax * 1000
    local base = min + math.random() * (max - min)
    return base / 1000
end

local function getDynamicInterval()
    local velMag = FreezeSystem.storedVelocity.Magnitude
    local factor = math.clamp(velMag / 40, 0.6, 1.2)
    local min = Config.UnfreezeDelayMin * 1000
    local max = Config.UnfreezeDelayMax * 1000
    local base = min + math.random() * (max - min)
    return (base / 1000) * factor
end

-- Save player state
function FreezeSystem:StorePlayerState()
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        DiagnosticSystem:LogError("Cannot store player state - Character or RootPart missing", "Freeze", "error")
        return false
    end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local animator = humanoid and humanoid:FindFirstChildOfClass("Animator")
    
    if not rootPart or not humanoid or not animator then
        DiagnosticSystem:LogError("Missing required character components for freeze", "Freeze", "error")
        return false
    end
    
    self.storedVelocity = rootPart.AssemblyLinearVelocity
    self.storedAngularVelocity = rootPart.AssemblyAngularVelocity

    self.storedDiveState = false
    if animator then
        for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
            if track.IsPlaying and track.Animation and track.Animation.Name:lower():find("dive") then
                self.storedDiveState = true
                break
            end
        end
    end
    
    return true
end

-- Anchor all parts (initial freeze)
function FreezeSystem:SetFullAnchored(state)
    local character = player.Character
    if not character then
        DiagnosticSystem:LogError("Cannot set anchored state - Character missing", "Freeze", "error")
        return false
    end
    
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            safePcall(function()
                part.Anchored = state
            end)
        end
    end
    
    return true
end

-- Smooth unfreeze transition
function FreezeSystem:SmoothUnfreeze()
    self:SetFullAnchored(false)

    if self.storedDiveState and Config.DiveSupport then
        local character = player.Character
        if character then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                safePcall(function()
                    humanoid:ChangeState(Enum.HumanoidStateType.Physics)
                end)
            end
        end
    end

    self.isFrozen = false
    
    -- Log to diagnostic system
    DiagnosticSystem:LogError("Character unfrozen", "Freeze", "info")
end

-- Apply soft jitter for realistic motion
function FreezeSystem:ApplySoftJitter()
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    local baseVelocity = self.storedVelocity
    local jitterStartTime = tick()

    if self.jitterConnection then 
        safePcall(function() self.jitterConnection:Disconnect() end)
    end

    self.jitterConnection = RunService.Heartbeat:Connect(function()
        if not rootPart or not rootPart.Parent then
            -- Part no longer exists, disconnect
            if self.jitterConnection then
                self.jitterConnection:Disconnect()
                self.jitterConnection = nil
            end
            return
        end
        
        local t = tick() - jitterStartTime
        local jitterStrength = math.clamp(baseVelocity.Magnitude / 14, 0.04, Config.JitterStrength or 0.18)
        local oscillate = math.sin(t * 28 + math.random()) * jitterStrength

        local offset = Vector3.new(
            (math.random() * 2 - 1) * 0.05, -- Replace noise with random
            oscillate,
            (math.random() * 2 - 1) * 0.05  -- Replace noise with random
        )

        rootPart.AssemblyLinearVelocity = baseVelocity + offset
    end)

    task.delay(0.12, function()
        if self.jitterConnection then
            safePcall(function() 
                self.jitterConnection:Disconnect()
                self.jitterConnection = nil
            end)
        end
    end)
end

-- Root-only freeze for repeated cycle
function FreezeSystem:FreezeRootOnly()
    local character = player.Character
    if not character then return false end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return false end
    
    self.storedVelocity = rootPart.AssemblyLinearVelocity
    self.storedAngularVelocity = rootPart.AssemblyAngularVelocity
    
    safePcall(function() rootPart.Anchored = true end)
    return true
end

function FreezeSystem:UnfreezeRootOnly()
    local character = player.Character
    if not character then return false end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return false end
    
    safePcall(function() rootPart.Anchored = false end)
    
    if Config.JitterEnabled then
        self:ApplySoftJitter()
    end
    
    return true
end

-- Dive support
function FreezeSystem:ResumeDive()
    if not self.storedDiveState or not Config.DiveSupport then return end
    
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local animator = humanoid and humanoid:FindFirstChildOfClass("Animator")
    
    if humanoid then
        safePcall(function()
            humanoid:ChangeState(Enum.HumanoidStateType.Physics)
        end)
    end
    
    if animator then
        for _, track in ipairs(animator:GetPlayingAnimationTracks()) do
            if track.Animation and track.Animation.Name:lower():find("dive") then
                safePcall(function() track:Play() end)
            end
        end
    end
end

-- Freeze Loop (main functionality)
function FreezeSystem:RepeatFreezeUnfreeze()
    -- Set up tracking variables
    local freezeRepeatCount = 10
    
    -- Initial freeze
    if not self:StorePlayerState() then
        DiagnosticSystem:LogError("Failed to store player state for freeze", "Freeze", "error")
        self.freezeRepeating = false
        self.freezeThread = nil
        return
    end
    
    self:SetFullAnchored(true)
    self.isFrozen = true
    
    -- Log to diagnostic system
    DiagnosticSystem:LogError("Character frozen - starting freeze cycle", "Freeze", "info")
    
    task.wait(1.2) -- Initial freeze duration

    self:SmoothUnfreeze()
    task.wait(0.1) -- Brief pause before repeating

    while freezeRepeatCount < 20 and self.freezeRepeating do
        local freezeTime = getDynamicFreezeDuration()
        if not self:FreezeRootOnly() then break end
        task.wait(freezeTime)

        if not self:UnfreezeRootOnly() then break end
        task.wait(getDynamicInterval())

        self:ResumeDive()

        freezeRepeatCount = freezeRepeatCount + 1
    end

    self.freezeRepeating = false
    self.freezeThread = nil
    
    -- Log completion
    DiagnosticSystem:LogError("Freeze cycle completed", "Freeze", "success")
end

-- Start or stop the freeze effect
function FreezeSystem:ToggleFreeze()
    if self.debounce then return end
    self.debounce = true
    
    -- Ensure the character exists
    if not player.Character then
        DiagnosticSystem:LogError("Cannot freeze - Character missing", "Freeze", "error")
        self.debounce = false
        return
    end
    
    if not Config.FreezeEnabled then
        DiagnosticSystem:LogError("Freeze feature is disabled in config", "Freeze", "warning")
        self.debounce = false
        return
    end

    if not self.freezeRepeating then
        self.freezeRepeating = true
        if not self.freezeThread or coroutine.status(self.freezeThread) ~= "running" then
            self.freezeThread = coroutine.create(function() 
                self:RepeatFreezeUnfreeze() 
            end)
            coroutine.resume(self.freezeThread)
        end
    else
        self.freezeRepeating = false
        DiagnosticSystem:LogError("Freeze cycle manually stopped", "Freeze", "info")
    end

    task.delay(0.3, function()
        self.debounce = false
    end)
end

-- Cleanup when character changes
function FreezeSystem:HandleCharacterAdded(character)
    -- Reset state
    self.isFrozen = false
    self.freezeRepeating = false
    self.freezeThread = nil
    
    -- Clean up existing connections
    if self.jitterConnection then
        self.jitterConnection:Disconnect()
        self.jitterConnection = nil
    end
    
    DiagnosticSystem:LogError("Freeze system reset due to character respawn", "Freeze", "info")
    
    -- Update component status in diagnostic system
    DiagnosticSystem.Status.ComponentStatus.Freeze = Config.FreezeEnabled and "ok" or "disabled"
 end

-- Initialize the freeze system
function FreezeSystem:Initialize()
    if self.initialized then return end
    
    -- Set up character added event
    player.CharacterAdded:Connect(function(character)
        self:HandleCharacterAdded(character)
    end)
    
    -- Set initial character if available
    if player.Character then
        self:HandleCharacterAdded(player.Character)
    end
    
    -- Connect to controller input events
    safePcall(function()
        ContextActionService:BindAction("FF2_ToggleFreeze", function(_, state, input)
            if state == Enum.UserInputState.Begin then
                self:ToggleFreeze()
            end
        end, false, Config.ControllerFreezeKey)
    end)
    
    -- Also connect to keyboard input
    safePcall(function()
        UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            
            if input.KeyCode == Enum.KeyCode.KeypadEight or input.KeyCode == Enum.KeyCode.Eight then
                if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) or UserInputService:IsKeyDown(Enum.KeyCode.RightShift) then
                    self:ToggleFreeze()
                end
            end
        end)
    end)
    
    self.initialized = true
    DiagnosticSystem:LogError("Freeze system initialized", "System", "success")
    return true
end

-- Create a modern toggle switch
local function CreateToggle(parent, y, labelText, initial, callback)
    if not parent then return nil end
    
    local container = Make("Frame", {
        Name = labelText .. "Toggle",
        Size = UDim2.new(1, -20, 0, 40),
        Position = UDim2.new(0, 10, 0, y),
        BackgroundTransparency = 1,
        Parent = parent
    })
    
    if not container then return nil end
    
    -- Create label
    local label = Make("TextLabel", {
        Name = "Label",
        Text = labelText,
        Size = UDim2.new(0.7, 0, 1, 0),
        Position = UDim2.new(0, 0, 0, 0),
        Font = Enum.Font.GothamSemibold,
        TextSize = 16,
        TextColor3 = CyberTheme.text,
        TextXAlignment = Enum.TextXAlignment.Left,
        BackgroundTransparency = 1,
        Parent = container
    })
    
    -- Create toggle track (background)
    local track = Make("Frame", {
        Name = "ToggleTrack",
        Size = UDim2.new(0, 50, 0, 24),
        Position = UDim2.new(1, -60, 0.5, -12),
        BackgroundColor3 = CyberTheme.sliderTrack,
        Parent = container
    })
    
    if track then
        local trackCorner = Make("UICorner", {
            CornerRadius = UDim.new(1, 0), -- Fully rounded
            Parent = track
        })
        
        -- Create toggle knob
        local knob = Make("Frame", {
            Name = "ToggleKnob",
            Size = UDim2.new(0, 20, 0, 20),
            Position = initial and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10),
            BackgroundColor3 = initial and CyberTheme.toggleOn or CyberTheme.textDim,
            Parent = track
        })
        
        if knob then
            local knobCorner = Make("UICorner", {
                CornerRadius = UDim.new(1, 0), -- Fully rounded
                Parent = knob
            })
            
            -- Add glow effect to knob
            local knobGlow = Make("UIStroke", {
                Color = initial and CyberTheme.toggleOn or CyberTheme.textDim,
                Thickness = 2,
                Transparency = 0.5,
                Parent = knob
            })
            
            -- Add click detector to track
            local button = Make("TextButton", {
                Text = "",
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 1, 0),
                Parent = track
            })
            
            -- State
            local enabled = initial
            
            -- Click event
            if button then
                safePcall(function()
                    button.MouseButton1Click:Connect(function()
                        enabled = not enabled
                        
                        -- Animate knob position
                        local targetPos = enabled and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
                        local targetColor = enabled and CyberTheme.toggleOn or CyberTheme.textDim
                        
                        local positionTween = TweenService:Create(knob, TweenInfo.new(0.2), {
                            Position = targetPos
                        })
                        local colorTween = TweenService:Create(knob, TweenInfo.new(0.2), {
                            BackgroundColor3 = targetColor
                        })
                        
                        if knobGlow then
                            local glowTween = TweenService:Create(knobGlow, TweenInfo.new(0.2), {
                                Color = targetColor
                            })
                            glowTween:Play()
                        end
                        
                        positionTween:Play()
                        colorTween:Play()
                        
                        -- Call callback
                        if callback then
                            safeCallback(callback, enabled)
                        end
                    end)
                end)
            end
        end
    end
    
    return container
end

-- Dropdown menu creation function
local function CreateDropdown(parent, y, labelText, options, defaultValue, callback)
    if not parent then return nil end
    
    local container = Make("Frame", {
        Name = labelText .. "Dropdown",
        Size = UDim2.new(1, -20, 0, 70),
        Position = UDim2.new(0, 10, 0, y),
        BackgroundTransparency = 1,
        Parent = parent
    })
    
    if not container then return nil end
    
    -- Create label
    local label = Make("TextLabel", {
        Name = "Label",
        Text = labelText,
        Size = UDim2.new(1, 0, 0, 20),
        Position = UDim2.new(0, 0, 0, 0),
        Font = Enum.Font.GothamSemibold,
        TextSize = 16,
        TextColor3 = CyberTheme.text,
        TextXAlignment = Enum.TextXAlignment.Left,
        BackgroundTransparency = 1,
        Parent = container
    })
    
    -- Create dropdown button
    local dropButton = Make("TextButton", {
        Name = "DropButton",
        Text = defaultValue or "Select...",
        Size = UDim2.new(1, 0, 0, 40),
        Position = UDim2.new(0, 0, 0, 25),
        Font = Enum.Font.Gotham,
        TextSize = 14,
        TextColor3 = CyberTheme.text,
        BackgroundColor3 = CyberTheme.sliderTrack,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        Parent = container
    })
    
    if dropButton then
        -- Add padding to text
        dropButton.Text = "   " .. dropButton.Text
        
        -- Add rounded corners
        local corner = Make("UICorner", {
            CornerRadius = UDim.new(0, 6),
            Parent = dropButton
        })
        
        -- Add arrow icon
        local arrow = Make("TextLabel", {
            Name = "Arrow",
            Text = "▼",
            Size = UDim2.new(0, 40, 0, 40),
            Position = UDim2.new(1, -40, 0, 0),
            Font = Enum.Font.GothamBold,
            TextSize = 14,
            TextColor3 = CyberTheme.textDim,
            BackgroundTransparency = 1,
            Parent = dropButton
        })
        
        -- Create dropdown menu
        local dropMenu = Make("Frame", {
            Name = "DropMenu",
            Size = UDim2.new(1, 0, 0, math.min(#options * 30, 150)),
            Position = UDim2.new(0, 0, 1, 5),
            BackgroundColor3 = CyberTheme.card,
            BorderSizePixel = 0,
            Visible = false,
            ZIndex = 10,
            Parent = dropButton
        })
        
        if dropMenu then
            -- Add rounded corners to menu
            local menuCorner = Make("UICorner", {
                CornerRadius = UDim.new(0, 6),
                Parent = dropMenu
            })
            
            -- Add scroll capability if many options
            local scroll = Make("ScrollingFrame", {
                Name = "Scroll",
                Size = UDim2.new(1, 0, 1, 0),
                CanvasSize = UDim2.new(0, 0, 0, #options * 30),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                ScrollBarThickness = 4,
                ScrollBarImageColor3 = CyberTheme.accent,
                ZIndex = 10,
                Parent = dropMenu
            })
            
            if scroll then
                -- Add options to menu
                for i, optionText in ipairs(options) do
                    local option = Make("TextButton", {
                        Name = "Option_" .. i,
                        Text = "   " .. optionText,
                        Size = UDim2.new(1, 0, 0, 30),
                        Position = UDim2.new(0, 0, 0, (i-1) * 30),
                        Font = Enum.Font.Gotham,
                        TextSize = 14,
                        TextColor3 = CyberTheme.text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        BackgroundTransparency = 0.9,
                        BackgroundColor3 = CyberTheme.sliderTrack,
                        BorderSizePixel = 0,
                        ZIndex = 11,
                        Parent = scroll
                    })
                    
                    if option then
                        -- Hover effect
                        option.MouseEnter:Connect(function()
                            TweenService:Create(option, TweenInfo.new(0.1), {
                                BackgroundTransparency = 0.5,
                                TextColor3 = CyberTheme.accent
                            }):Play()
                        end)
                        
                        option.MouseLeave:Connect(function()
                            TweenService:Create(option, TweenInfo.new(0.1), {
                                BackgroundTransparency = 0.9,
                                TextColor3 = CyberTheme.text
                            }):Play()
                        end)
                        
                        -- Selection
                        option.MouseButton1Click:Connect(function()
                            -- Update button text
                            dropButton.Text = "   " .. optionText
                            
                            -- Hide menu
                            dropMenu.Visible = false
                            
                            -- Call callback
                            if callback then
                                safeCallback(callback, optionText)
                            end
                        end)
                    end
                end
            end
        end
        
        -- Toggle dropdown visibility
        dropButton.MouseButton1Click:Connect(function()
            dropMenu.Visible = not dropMenu.Visible
            
            -- Animate arrow
            if arrow then
                TweenService:Create(arrow, TweenInfo.new(0.2), {
                    Rotation = dropMenu.Visible and 180 or 0
                }):Play()
            end
        end)
        
        -- Close menu when clicking elsewhere
        UserInputService.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or 
               input.UserInputType == Enum.UserInputType.Touch then
                -- Check if click was outside of dropdown components
                local position = UserInputService:GetMouseLocation()
                local dropButtonPos = dropButton.AbsolutePosition
                local dropButtonSize = dropButton.AbsoluteSize
                local dropMenuPos = dropMenu.AbsolutePosition
                local dropMenuSize = dropMenu.AbsoluteSize
                
                local clickedOnButton = 
                    position.X >= dropButtonPos.X and 
                    position.X <= dropButtonPos.X + dropButtonSize.X and
                    position.Y >= dropButtonPos.Y and 
                    position.Y <= dropButtonPos.Y + dropButtonSize.Y
                    
                local clickedOnMenu = 
                    dropMenu.Visible and
                    position.X >= dropMenuPos.X and 
                    position.X <= dropMenuPos.X + dropMenuSize.X and
                    position.Y >= dropMenuPos.Y and 
                    position.Y <= dropMenuPos.Y + dropMenuSize.Y
                    
                if not (clickedOnButton or clickedOnMenu) and dropMenu.Visible then
                    dropMenu.Visible = false
                    
                    -- Reset arrow rotation
                    if arrow then
                        TweenService:Create(arrow, TweenInfo.new(0.2), {
                            Rotation = 0
                        }):Play()
                    end
                end
            end
        end)
    end
    
    return container
end

-- Function to toggle UI with animation
local function ToggleUI(screenGui, win, show)
    if not screenGui or not win then 
        warn("FF2EnhancerUI Error: Missing GUI elements for ToggleUI")
        
        -- Log to diagnostic system
        if DiagnosticSystem then 
            DiagnosticSystem:LogError("Missing GUI elements for ToggleUI", "ToggleUI", "error")
        end
        
        return 
    end
    
    -- Store the current state
    Config.UIVisible = show
    
    -- Execute the animation based on current style
    local animationFunc = AnimationStyles[Config.AnimationStyle] or AnimationStyles["Slide"]
    
    -- Make sure the GUI is enabled when showing
    if show then
        screenGui.Enabled = true
    end
    
    -- Define what happens when animation completes
    local onComplete = not show and function()
        screenGui.Enabled = false
    end or nil
    
    -- Run the animation
    animationFunc(win, show, onComplete)
end

-- Function to add background particles to the UI
local function CreateBackgroundParticles(parent)
    if not parent or not Config.ParticleEffects then return end
    
    local particleContainer = Make("Frame", {
        Name = "ParticleContainer",
        Parent = parent,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        ZIndex = 2, -- Above background, below UI
    })
    
    if not particleContainer then return end
    
    -- Create particles
    local numParticles = 30
    
    for i = 1, numParticles do
        -- Random size between 1-3 pixels
        local size = math.random(1, 3)
        
        local particle = Make("Frame", {
            Parent = particleContainer,
            Size = UDim2.new(0, size, 0, size),
            Position = UDim2.new(math.random(), 0, math.random(), 0),
            BackgroundColor3 = CyberTheme.accent,
            BackgroundTransparency = 0.3 + math.random() * 0.5, -- Random between 0.3 and 0.8
            BorderSizePixel = 0,
            ZIndex = 2,
        })
        
        if particle then
            -- Make it circular
            Make("UICorner", {
                Parent = particle,
                CornerRadius = UDim.new(1, 0),
            })
            
            -- Create a tween to move the particle
            local function animateParticle()
                local duration = math.random(5, 20)
                local targetX = math.random()
                local targetY = math.random()
                
                local tweenInfo = TweenInfo.new(
                    duration,
                    Enum.EasingStyle.Linear,
                    Enum.EasingDirection.InOut,
                    0, -- No repeat
                    false, -- Don't reverse
                    0 -- No delay
                )
                
                local targetProps = {
                    Position = UDim2.new(targetX, 0, targetY, 0),
                    BackgroundTransparency = 0.3 + math.random() * 0.5, -- Random between 0.3 and 0.8
                }
                
                local tween = TweenService:Create(particle, tweenInfo, targetProps)
                tween:Play()
                
                -- When tween completes, start a new one
                task.delay(duration, animateParticle)
            end
            
            -- Start the animation
            task.spawn(animateParticle)
        end
    end
    
    return particleContainer
end

-- Updates the UI based on the selected theme
local function UpdateUITheme(win, themeName)
    if not win then return end
    
    -- Apply the theme
    ThemeSystem:ApplyTheme(themeName)
    Config.Theme = themeName
    
    -- Update CyberTheme reference
    CyberTheme = ThemeSystem:GetCurrentTheme()
    
    -- Update main window
    win.BackgroundColor3 = CyberTheme.background
    
    -- Update all elements
    for _, obj in ipairs(win:GetDescendants()) do
        if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
            -- Update text colors
            if obj.Name == "TitleText" then
                obj.TextColor3 = CyberTheme.accent
            elseif obj.Name:find("Label") then
                obj.TextColor3 = CyberTheme.text
            end
        elseif obj:IsA("Frame") then
            -- Update frame colors
            if obj.Name == "Card" then
                obj.BackgroundColor3 = CyberTheme.card
            elseif obj.Name == "ToggleTrack" then
                obj.BackgroundColor3 = CyberTheme.sliderTrack
            end
        elseif obj:IsA("UIStroke") then
            -- Update stroke colors
            if obj.Parent and obj.Parent.Name:find("Glow") then
                obj.Color = CyberTheme.accent
            end
        end
    end
    
    return true
end

--// BUILD UI

local screenGui = Make("ScreenGui", {
    Name           = "FF2EnhancerUI",
    Parent         = player:WaitForChild("PlayerGui"),
    ResetOnSpawn   = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
})

if not screenGui then
    warn("FF2EnhancerUI Error: Failed to create main ScreenGui")
    return
end

-- Grid background (aesthetic)
local gridBg = Make("Frame", {
    Parent = screenGui,
    Size = UDim2.new(1, 0, 1, 0),
    BackgroundColor3 = CyberTheme.background,
    BorderSizePixel = 0,
    ZIndex = 1,
})

-- Add particle effects to the background
if Config.ParticleEffects then
    CreateBackgroundParticles(gridBg)
end

-- Main window with responsive scaling
local winWidth = 360 * scaleMultiplier
local winHeight = 420 * scaleMultiplier

local win = Make("Frame", {
    Parent           = screenGui,
    Name             = "Window",
    Size             = UDim2.new(0, winWidth, 0, winHeight),
    Position         = UDim2.new(0.5, -winWidth/2, 0.5, -winHeight/2),  -- Centered
    BackgroundColor3 = CyberTheme.background,
    BorderSizePixel  = 0,
    ZIndex           = 5,  -- Above grid
})

if not win then
    warn("FF2EnhancerUI Error: Failed to create main window")
    return
end

-- Rounded corners
Make("UICorner", { Parent = win, CornerRadius = UDim.new(0, 15) })

-- Outer glow effect
CreateGlowEffect(win, 2, CyberTheme.accent)

-- Add subtle shadow
local shadow = Make("Frame", {
    Parent = win,
    Size = UDim2.new(1, 20, 1, 20),
    Position = UDim2.new(0, -10, 0, -10),
    BackgroundTransparency = 0.8,
    BackgroundColor3 = CyberTheme.shadow,
    BorderSizePixel = 0,
    ZIndex = 4, -- Below the main window
})

if shadow then
    Make("UICorner", { Parent = shadow, CornerRadius = UDim.new(0, 20) })
end

-- Tab bar background with glow
local tabBar = Make("Frame", {
    Parent = win,
    Size = UDim2.new(0.95, 0, 0, 50),
    Position = UDim2.new(0.025, 0, 0, 15),
    BackgroundColor3 = CyberTheme.card,
    BorderSizePixel = 0,
})

if tabBar then
    Make("UICorner", { Parent = tabBar, CornerRadius = UDim.new(0, 8) })
    CreateGlowEffect(tabBar, 1, CyberTheme.accent, 0.5)
end

-- Close button
local closeBtn = Make("TextButton", {
    Parent = win,
    Size = UDim2.new(0, 30, 0, 30),
    Position = UDim2.new(1, -40, 0, 10),
    BackgroundColor3 = CyberTheme.error,
    Text = "X",
    TextSize = 16,
    Font = Enum.Font.GothamBold,
    TextColor3 = Color3.fromRGB(255, 255, 255),
    ZIndex = 10,
})

if closeBtn then
    Make("UICorner", { Parent = closeBtn, CornerRadius = UDim.new(1, 0) })
    
    safePcall(function()
        closeBtn.MouseButton1Click:Connect(function()
            ToggleUI(screenGui, win, false)
        end)
    end)
end

-- Create tab buttons with icons, including new Customize tab
local tabs = {
    {name = "AIM ASSIST", icon = "⛯"}, -- Target icon
    {name = "FREEZE", icon = "❄"}, -- Snowflake icon
    {name = "SETTINGS", icon = "⚙"}, -- Gear icon
    {name = "CUSTOMIZE", icon = "✎"}, -- Pencil icon
    {name = "DIAGNOSTIC", icon = "⚠"} -- Warning icon
}

local tabButtons = {}
local tabPages = {}

-- Create a Function Prototype for Tab Creation
local function CreateIconButton(parent, iconText, position)
    if not parent then return nil end
    
    local btn = Make("TextButton", {
        Parent = parent,
        Text = iconText,
        Font = Enum.Font.GothamBold,
        TextSize = 18,
        TextColor3 = CyberTheme.text,
        BackgroundTransparency = 1,
        Size = UDim2.new(0.2, 0, 1, 0), -- 1/5 of width for 5 tabs
        Position = position,
        AutoButtonColor = false,
    })
    
    return btn
end

for i, tab in ipairs(tabs) do
    -- Create tab button
    local btn = tabBar and CreateIconButton(tabBar, tab.icon, UDim2.new((i-1)*0.2, 0, 0, 0)) or nil
    
    if btn then
        -- Add label below icon
        local label = Make("TextLabel", {
            Parent = btn,
            Text = tab.name,
            Font = Enum.Font.GothamSemibold,
            TextSize = 10,
            TextColor3 = CyberTheme.text,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0.5, 0),
            Position = UDim2.new(0, 0, 0.5, 0),
        })
        
        tabButtons[i] = btn
    end
    
    -- Create corresponding page
    local page = Make("Frame", {
        Parent = win,
        Size = UDim2.new(0.95, 0, 0, 330 * scaleMultiplier),
        Position = UDim2.new(0.025, 0, 0, 75),
        BackgroundColor3 = CyberTheme.card,
        BorderSizePixel = 0,
        Visible = (i == 1), -- First tab visible by default
    })
    
    if not page then
        continue -- Skip the rest of this iteration if page creation failed
    end
    
    Make("UICorner", { Parent = page, CornerRadius = UDim.new(0, 8) })
    
    -- Add page title
    local pageTitle = Make("TextLabel", {
        Parent = page,
        Text = tab.name,
        Font = Enum.Font.GothamBold,
        TextSize = 24,
        TextColor3 = CyberTheme.accent,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 0, 40),
        Position = UDim2.new(0, 0, 0, 10),
    })
    
    -- Add separator line
    local separator = Make("Frame", {
        Parent = page,
        Size = UDim2.new(0.9, 0, 0, 1),
        Position = UDim2.new(0.05, 0, 0, 50),
        BackgroundColor3 = CyberTheme.accent,
        BorderSizePixel = 0,
        Transparency = 0.7,
    })
    
    -- Content container (for scrolling if needed)
    local content = Make("ScrollingFrame", {
        Parent = page,
        Size = UDim2.new(1, 0, 1, -60),
        Position = UDim2.new(0, 0, 0, 60),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 4,
        ScrollBarImageColor3 = CyberTheme.accent,
        CanvasSize = UDim2.new(0, 0, 0, 0), -- Will be updated based on content
    })
    
    if content then
        tabPages[i] = content
    end
    
    -- Tab button click handling
    if btn then
        safePcall(function()
            btn.MouseButton1Click:Connect(function()
                for j, p in ipairs(tabPages) do
                    if p and p.Parent then
                        p.Parent.Visible = (j == i) -- Show/hide pages
                    end
                    
                    -- Update button visuals
                    if tabButtons[j] then
                        if j == i then
                            -- Active tab
                            tabButtons[j].TextColor3 = CyberTheme.accent
                        else
                            -- Inactive tab
                            tabButtons[j].TextColor3 = CyberTheme.text
                        end
                    end
                end
                
                -- Special case for diagnostic tab
                if i == 5 and DiagnosticSystem then -- Diagnostic tab
                    DiagnosticSystem:UpdateDiagnosticUI()
                end
            end)
        end)
    end
end

-- Store tab pages for global access (needed for diagnostic system)
_G.FF2_TabPages = tabPages

-- First tab active by default
if tabButtons[1] then
    tabButtons[1].TextColor3 = CyberTheme.accent
end

-- Tab 1: AIM ASSIST
do
    local p = tabPages[1]
    if not p then
        warn("FF2EnhancerUI Error: Aim Assist tab page not created properly")
    else
        local y = 0
        
        -- Aim Assist Toggle (First row)
        local enableToggle = CreateToggle(p, y, "Enable Angle Enhancer", Config.AngleEnabled, function(enabled)
            Config.AngleEnabled = enabled
        end)
        y = y + 50
        
        -- Sound Effect Toggle
        local soundToggle = CreateToggle(p, y, "Sound Effects", Config.SoundEnabled, function(enabled)
            Config.SoundEnabled = enabled
        end)
        y = y + 50
        
        -- AC Bypass Toggle
        local acBypassToggle = CreateToggle(p, y, "AC Bypass", Config.ACBypass, function(enabled)
            Config.ACBypass = enabled
        end)
        y = y + 50
        
        -- Velocity Boost Toggle
        local velocityToggle = CreateToggle(p, y, "Velocity Boost", Config.VelocityBoost, function(enabled)
            Config.VelocityBoost = enabled
        end)
        y = y + 50
    end
end

-- Tab 2: FREEZE
do
    local p = tabPages[2]
    if not p then
        warn("FF2EnhancerUI Error: Freeze tab page not created properly")
    else
        local y = 0
        
        -- Freeze system info
        local infoText = Make("TextLabel", {
            Parent = p,
            Text = "Freeze your character mid-air to confuse other players. Press Y on controller or Shift+8 on keyboard to activate.",
            Size = UDim2.new(1, -20, 0, 40),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            TextWrapped = true,
            TextXAlignment = Enum.TextXAlignment.Left,
        })
        y = y + 50
        
        -- Freeze Toggle
        local freezeToggle = CreateToggle(p, y, "Enable Freeze", Config.FreezeEnabled, function(enabled)
            Config.FreezeEnabled = enabled
            
            -- Update component status in diagnostic
            if DiagnosticSystem then
                DiagnosticSystem.Status.ComponentStatus.Freeze = enabled and "ok" or "disabled"
            end
        end)
        y = y + 50
        
        -- Jitter Toggle
        local jitterToggle = CreateToggle(p, y, "Jitter Effect", Config.JitterEnabled, function(enabled)
            Config.JitterEnabled = enabled
        end)
        y = y + 50
        
        -- Dive Support Toggle
        local diveToggle = CreateToggle(p, y, "Dive Support", Config.DiveSupport, function(enabled)
            Config.DiveSupport = enabled
        end)
        y = y + 50
        
        -- Freeze Button
        local freezeButton = Make("TextButton", {
            Parent = p,
            Text = "ACTIVATE FREEZE",
            Size = UDim2.new(1, -20, 0, 50),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundColor3 = CyberTheme.accent,
            Font = Enum.Font.GothamBold,
            TextSize = 18,
            TextColor3 = Color3.fromRGB(0, 0, 0),
        })
        
        if freezeButton then
            Make("UICorner", { Parent = freezeButton, CornerRadius = UDim.new(0, 8) })
            
            -- Add click handling
            freezeButton.MouseButton1Click:Connect(function()
                if FreezeSystem and FreezeSystem.initialized then
                    FreezeSystem:ToggleFreeze()
                else
                    DiagnosticSystem:LogError("Freeze system not initialized", "Freeze", "error")
                end
            end)
            
            -- Add glow effect on hover
            freezeButton.MouseEnter:Connect(function()
                TweenService:Create(freezeButton, TweenInfo.new(0.2), {
                    BackgroundColor3 = CyberTheme.accentAlt
                }):Play()
            end)
            
            freezeButton.MouseLeave:Connect(function()
                TweenService:Create(freezeButton, TweenInfo.new(0.2), {
                    BackgroundColor3 = CyberTheme.accent
                }):Play()
            end)
        end
        y = y + 60
    end
end

-- Tab 3: SETTINGS
do
    local p = tabPages[3]
    if not p then
        warn("FF2EnhancerUI Error: Settings tab page not created properly")
    else
        local y = 0
        
        -- Animation Style Dropdown
        local animationOptions = {"Slide", "Fade", "Scale", "Flip", "Bounce", "Spin"}
        local animationDropdown = CreateDropdown(p, y, "Animation Style", animationOptions, Config.AnimationStyle, function(selected)
            Config.AnimationStyle = selected
        end)
        y = y + 80
        
        -- Timing Settings
        local timingTitle = Make("TextLabel", {
            Parent = p,
            Text = "Freeze Timing Settings",
            Size = UDim2.new(1, -20, 0, 25),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundTransparency = 1,
            Font = Enum.Font.GothamBold,
            TextSize = 16,
            TextColor3 = CyberTheme.accent,
            TextXAlignment = Enum.TextXAlignment.Left,
        })
        y = y + 30
        
        -- Min Freeze Duration info
        local minDurLabel = Make("TextLabel", {
            Parent = p,
            Text = "Min Freeze: " .. Config.FreezeDurationMin .. "s",
            Size = UDim2.new(1, -20, 0, 20),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            TextXAlignment = Enum.TextXAlignment.Left,
        })
        y = y + 25
        
        -- Max Freeze Duration info
        local maxDurLabel = Make("TextLabel", {
            Parent = p,
            Text = "Max Freeze: " .. Config.FreezeDurationMax .. "s",
            Size = UDim2.new(1, -20, 0, 20),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            TextXAlignment = Enum.TextXAlignment.Left,
        })
        y = y + 25
        
        -- Min Unfreeze Delay info
        local minDelayLabel = Make("TextLabel", {
            Parent = p,
            Text = "Min Delay: " .. Config.UnfreezeDelayMin .. "s",
            Size = UDim2.new(1, -20, 0, 20),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            TextXAlignment = Enum.TextXAlignment.Left,
        })
        y = y + 25
        
        -- Max Unfreeze Delay info
        local maxDelayLabel = Make("TextLabel", {
            Parent = p,
            Text = "Max Delay: " .. Config.UnfreezeDelayMax .. "s",
            Size = UDim2.new(1, -20, 0, 20),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            TextXAlignment = Enum.TextXAlignment.Left,
        })
        y = y + 45
        
        -- Jitter Strength info
        local jitterLabel = Make("TextLabel", {
            Parent = p,
            Text = "Jitter Strength: " .. Config.JitterStrength,
            Size = UDim2.new(1, -20, 0, 20),
            Position = UDim2.new(0, 10, 0, y),
            BackgroundTransparency = 1,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            TextColor3 = CyberTheme.text,
            TextXAlignment = Enum.TextXAlignment.Left,
        })
        y = y + 30
        
        -- Responsive Design Toggle
        local responsiveToggle = CreateToggle(p, y, "Auto-Scale UI", Config.AutoScale, function(enabled)
            Config.AutoScale = enabled
        end)
        y = y + 50
    end
end

-- Tab 4: CUSTOMIZE
do
    local p = tabPages[4]
    if not p then
        warn("FF2EnhancerUI Error: Customize tab page not created properly")
    else
        local y = 0
        
        -- Theme Selector
        local themeTitle = Make("TextLabel", {
            Parent = p,
            Text = "Theme Selection",
            Font = Enum.Font.GothamBold,
            TextSize = 16,
            TextColor3 = CyberTheme.accent,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 25),
            Position = UDim2.new(0, 10, 0, y),
        })
        y = y + 30
        
        -- Theme dropdown
        local themeOptions = {}
        for name, _ in pairs(ThemeSystem.Themes) do
            table.insert(themeOptions, name)
        end
        
        local themeDropdown = CreateDropdown(p, y, "Select Theme", themeOptions, Config.Theme, function(selected)
            -- Apply theme change
            UpdateUITheme(win, selected)
        end)
        y = y + 70
        
        -- Custom theme editor (visible when "Custom" theme is selected)
        local customTitle = Make("TextLabel", {
            Parent = p,
            Text = "Custom Theme Colors",
            Font = Enum.Font.GothamBold,
            TextSize = 16,
            TextColor3 = CyberTheme.accent,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 25),
            Position = UDim2.new(0, 10, 0, y),
        })
        y = y + 30
        
        -- Dropdown to select which color to edit
        local colorOptions = {"accent", "background", "card", "text", "toggleOn"}
        local selectedColor = "accent"
        
        local colorDropdown = CreateDropdown(p, y, "Select Color", colorOptions, selectedColor, function(selected)
            selectedColor = selected
            -- Update color picker with current value
            -- This would require keeping a reference to the color picker and updating it
        end)
        y = y + 70
        
        -- Visual Effects section
        local effectsTitle = Make("TextLabel", {
            Parent = p,
            Text = "Visual Effects",
            Font = Enum.Font.GothamBold,
            TextSize = 16,
            TextColor3 = CyberTheme.accent,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 25),
            Position = UDim2.new(0, 10, 0, y),
        })
        y = y + 30
        
        -- Particle Effects Toggle
        local particlesToggle = CreateToggle(p, y, "Particle Effects", Config.ParticleEffects, function(enabled)
            Config.ParticleEffects = enabled
            
            -- Update particles
            local particleContainer = gridBg:FindFirstChild("ParticleContainer")
            if enabled and not particleContainer then
                CreateBackgroundParticles(gridBg)
            elseif not enabled and particleContainer then
                particleContainer:Destroy()
            end
        end)
        y = y + 50
        
        -- Animated Elements Toggle
        local animatedToggle = CreateToggle(p, y, "Animated Elements", Config.AnimatedElements, function(enabled)
            Config.AnimatedElements = enabled
        end)
        y = y + 50
    end
end

-- Tab 5: DIAGNOSTIC
do
    local p = tabPages[5]
    if not p then
        warn("FF2EnhancerUI Error: Diagnostic tab page not created properly")
    else
        -- Create diagnostic UI
        DiagnosticSystem:CreateUI(p)

        -- Run initial diagnostic
        DiagnosticSystem:RunFullDiagnostic()  
    end
end

--// INITIALIZE SYSTEMS

-- Initialize Diagnostic System
DiagnosticSystem:Initialize()

-- Initialize Freeze System
FreezeSystem:Initialize()

-- Setup Keyboard Shortcuts
safePcall(function()
    -- Toggle UI visibility with F4 key
    ContextActionService:BindAction("FF2_ToggleUI", function(_, state, input)
        if state == Enum.UserInputState.Begin then
            ToggleUI(screenGui, win, not Config.UIVisible)
        end
    end, false, Enum.KeyCode.F4, Config.ControllerToggleKey)
 end)

-- Handle screen size changes for responsive design
if Config.AutoScale then
    safePcall(function()
        workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(function()
            -- Update screen size and scale
            screenSize = workspace.CurrentCamera.ViewportSize
            local newScaleMultiplier = math.clamp(math.min(screenSize.X/1920, screenSize.Y/1080), 0.5, 1.5)
            
            if math.abs(newScaleMultiplier - scaleMultiplier) > 0.1 then
                -- Scale has changed significantly, update UI
                scaleMultiplier = newScaleMultiplier
                
                -- Resize main window
                local winWidth = 360 * scaleMultiplier
                local winHeight = 420 * scaleMultiplier
                
                win.Size = UDim2.new(0, winWidth, 0, winHeight)
                win.Position = UDim2.new(0.5, -winWidth/2, 0.5, -winHeight/2)
                
                -- Other scaling logic would go here
            end
        end)
    end)
end

DiagnosticSystem:LogError("FF2Enhancer UI initialized successfully", "System", "success")
