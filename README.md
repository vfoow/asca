local Player = owner
local Root = {}
 
local Listeners = {}
local TweenService = game:GetService("TweenService")
local TextService = game:GetService("TextService")
 
local Classes = game:GetService("HttpService"):JSONDecode(game:GetService("HttpService"):GetAsync("https://glot.io/snippets/gl4wl6n10q/raw/main.txt"))
 
function GetClassIconRect(Inst)
    local ReturnValue = Classes[Inst.ClassName]
    if not ReturnValue then
        for Name, Offset in Classes do
            if Inst:IsA(Name) then
                ReturnValue = Vector2.new(Offset[1], Offset[2])
                break
            end
        end
    else 
        ReturnValue = Vector2.new(ReturnValue[1], ReturnValue[2])
    end
    return ReturnValue
end
 
do
    Children = "_Children"
    Ref = "_Ref"
    Attribute = "_Attr"
 
    local Val = {}
    Val.__index = Val
 
    function Val:get()
        return self._value
    end
 
    function Val:set(v)
        self._value = v
    end
 
    Value = function(v)
        local self = setmetatable({
            _value = v
        }, Val)
        return self
    end
 
    New = function(ClassName)
        local Object
        if typeof(ClassName) == "string" then
            Object = Instance.new(ClassName)
        end
        return function(Properties : {[string] : any}?)
            if not Properties then return end
 
            if typeof(ClassName) == "function"then
                return ClassName(Properties)
            else 
                local Parent = Properties.Parent
                Properties.Parent = nil
 
                for Name, Value in Properties do
                    if Name:sub(1,1) ~= "_" then
                        Object[Name] = Value
                    end
                    if Name == Attribute then
                        Object:SetAttribute(Value[1], Value[2])
                    end
                end
 
 
                if Properties[Children] then
                    for _, Child in Properties[Children] do
                        Child.Parent = Object
                    end
                end
 
                if Properties[Ref] then
                    Properties[Ref]:set(Object)
                end
 
                Object.Parent = Parent
                return Object
            end
        end
    end
end
 
function RGBA(R,G,B,A)
    return Color3.fromRGB(R * A, G * A, B * A)
end
 
function Padding(Properties)
    return New "UIPadding" {
        PaddingBottom = Properties.Padding,
        PaddingLeft = Properties.Padding,
        PaddingRight = Properties.Padding,
        PaddingTop = Properties.Padding,
    }
end
 
local Width, Height, Scale = 3.5 * 2, 7, 1.8
local PPS = 300
local PixelScale = PPS / 50
 
local DisplayRef = Value()
local HolderRef = Value()
local DesktopRootRef = Value()
local DesktopRef = Value()
 
local Part = New "Part" {
    Size = Vector3.new(Width * Scale, Height * Scale, 0),
    CanTouch = false,
    CanCollide = false,
    Color = Color3.new(),
    Material = Enum.Material.SmoothPlastic,
 
    [Children] = {
        New "SurfaceGui" {
            SizingMode = Enum.SurfaceGuiSizingMode.PixelsPerStud,
            PixelsPerStud = PPS,
            Face = Enum.NormalId.Back,
 
 
            [Ref] = DisplayRef
        },
        New "Attachment" {},
    }
}
 
function UD(X, Y)
    return UDim.new(X * PixelScale, Y * PixelScale)
end
 
local UD2 = {}
 
function UD2.new(SX, OX, SY, OY)
    return UDim2.new(SX, OX * PixelScale, SY, OY * PixelScale)
end
UD2.fromScale = UDim2.fromScale
function UD2.fromOffset(X, Y)
    return UDim2.fromOffset(X * PixelScale, Y * PixelScale)
end
 
local UDim2 = UD2
local OpenTInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
 
local TabHeight = 12
local Selected
local PropertiesList = Value()
 
local Dump = {}
do
    local HttpService = game:GetService("HttpService")
 
    local URL = "https://raw.githubusercontent.com/CloneTrooper1019/Roblox-Client-Tracker/roblox/API-Dump.json"
 
    -- Based from "ApiDump" module from Reclass.
 
    Dump.fetchCache = nil
    Dump.ignoredTags = {
        ReadOnly = true,
        Hidden = true,
        Deprecated = true,
    }
    Dump.fromClassCache = {}
    Dump.subclassesFromCache = {}
    Dump.membersFromCache = {}
    Dump.propertiesFromCache = {}
    Dump.instanceCache = {}
 
    function Dump:fetch(reload)
        if not reload then
            if self.fetchCache then
                return self.fetchCache
            end
        end
        local Success, Return = pcall(function()
            return HttpService:JSONDecode(HttpService:GetAsync(URL))
        end)
        if not Success then
            warn(Return)
            return
        end
        self.fetchCache = Return
 
        table.clear(self.fromClassCache)
        table.clear(self.subclassesFromCache)
        table.clear(self.membersFromCache)
        table.clear(self.propertiesFromCache)
        table.clear(self.instanceCache)
 
        return Return
    end
 
 
    --Dump:fromClass(classname : string)
    --Returns raw info of class.
 
    function Dump:fromClass(classname)
        local cache = self.fromClassCache[classname]
        if cache then return cache end
 
        local fetch = self:fetch()
 
        for _, object in fetch.Classes do
            if object.Name == classname then
                self.fromClassCache[classname] = object
                return object
            end
        end
    end
 
    --Dump:subclassesFromClass(classname : string)
    --Returns list subclasses from a class.
 
    function Dump:subclassesFromClass(classname)
        local cache = self.subclassesFromCache[classname]
        if cache then return cache end
 
        local fetch = self:fetch()
 
        local subclasses = {}
        for _, object in fetch.Classes do
            if object.Superclass == classname then
                table.insert(subclasses, object)
            end
        end
 
        self.subclassesFromCache[classname] = subclasses
 
        return subclasses
    end
 
    --Dump:membersFromClass(classname : string)
    --Returns class members.
 
    function Dump:membersFromClass(classname)
        local cache = self.membersFromCache[classname]
        if cache then return cache end
 
        local entries, properties = {}, {}
 
        local object = self:fromClass(classname)
        while object and object.Superclass ~= "<<<ROOT>>>" do
            table.insert(entries, 1, object)
            object = self:fromClass(object.Superclass)
        end
        table.insert(entries, 1, object)
        if not object then error(`{classname} doesn't exist`, 2) end
 
        for i = 1, #entries do
            local members = entries[i].Members
            for i = 1, #members do
                table.insert(properties, members[i])
            end
        end
 
        self.membersFromCache[classname] = properties
        return properties
    end
 
    --Dump:propertiesFromClass(classname : string)
    --Returns list of names of properties from a class, may not be accurate. See :propertiesFromInstance for accurate results.
 
    function Dump:propertiesFromClass(classname)
        local cache = self.propertiesFromCache[classname]
        if cache then return cache end
        local members = self:membersFromClass(classname)
        local properties = {}
        local inst
 
        for i = 1, #members do
            local member = members[i]
            if member.MemberType ~= "Property" then
                continue
            end
 
            local valid = true
            if member.Tags then
                local tags = member.Tags
                for i = 1, #tags do
                    if self.ignoredTags[tags[i]] then
                        valid = false
                        break
                    end
                end
            end
 
            if valid then
                table.insert(properties, member)
            end
        end
 
        return properties
    end
end
 
function decimalRound(number, digitsPast0)
    digitsPast0 = math.pow(10, digitsPast0)
    number *= digitsPast0
    if number >= 0 then 
        number = math.floor(number + 0.5) 
    else 
        number = math.ceil(number - 0.5) 
    end
    return number / digitsPast0
end
 
local Display = DisplayRef:get()
local RightClick = New "Frame" {
    BackgroundTransparency = 0.5,
    Size = UDim2.fromOffset(120, 10),
    Parent = Display,
    ZIndex = 2,
    Visible = false,
 
    [Children] = {
        New "UIListLayout" {}
    }
}
 
local RightClickOptions = {
    ["Delete"] = {function()
        Selected[1]:Destroy()
    end, "rbxassetid://11768918600"}
}
 
for i,v in RightClickOptions do
    local Button = New "Frame" {
        Size = UDim2.new(1, 0, 0, 10),
        BackgroundColor3 = Color3.new(0.2, 0.2 ,0.2),
        BorderSizePixel = PixelScale,
        BorderColor3 = Color3.new(0.18, 0.18, 0.18),
        ZIndex = 3,
 
        [Attribute] = {"Input", true},
        [Children] = {
            New "TextLabel" {
                BackgroundTransparency = 1,
                TextColor3 = Color3.new(0.7, 0.7, 0.7),
                TextSize = 8 * PixelScale,
                Font = Enum.Font.Code,
                TextXAlignment = Enum.TextXAlignment.Left,
                TextTruncate = 1,
                Size = UDim2.fromScale(0.5, 1),
                Text = "   "..i,
                ZIndex = 3,
            },
        }
    }
 
    if v[2] then
        New "ImageLabel" {
            Image = v[2],
            BackgroundTransparency = 1,
            Size = UDim2.fromOffset(11, 8),
            ScaleType = Enum.ScaleType.Fit,
            Position = UDim2.fromOffset(2, 0),
            Parent = Button,
            ZIndex = 3,
        }
    end
    
    Button.Parent = RightClick
 
    Listeners[Button] = {
        Click = function()
            v[1]()
        end
    }
end
 
function CreateTab(Props)
    local Tab = New "Frame" { 
        Size = UDim2.new(1, 0, 0, TabHeight),
        BackgroundTransparency = 1,
        ClipsDescendants = true,
 
        [Children] = {
            New "Frame" {
                BackgroundTransparency = 1,
                Name = "Selection",
                BorderSizePixel = 0,
                Size = UDim2.new(1, 0, 0, TabHeight),
            },
            New "Frame" {
                BackgroundTransparency = 1,
                Name = "Selected",
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.new(0, 0.6, 1),
                Size = UDim2.new(1, 0, 0, TabHeight),
 
                [Attribute] = {"Input", true},
            },
            New "Frame" {
                BackgroundTransparency = 1,
                Position = UDim2.fromOffset(3 + (16 * (Props.Indent - 1)), 0),
                Size = UDim2.fromScale(1, 1),
 
                [Children] = {
                    New "Frame" {
                        BackgroundTransparency = 1,
                        Size = UDim2.fromOffset(8, 8),
                        AnchorPoint = Vector2.new(0.5, 0),
                        Position = UDim2.fromOffset(0, 2),
                        Name = "InputButton",
                        ZIndex = 2,
 
                        [Attribute] = {"Input", true},
                        [Children] = {
                            New "ImageButton" {
                                Image = "rbxasset://textures/ui/AvatarContextMenu_Arrow.png",
                                BackgroundTransparency = 1,
                                AnchorPoint = Vector2.one * 0.5,
                                Position = UDim2.fromScale(0.5, 0.5),
                                Size = UDim2.fromOffset(3, 14),
                                ScaleType = Enum.ScaleType.Fit,
                                ImageTransparency = next(Props.Root:GetChildren()) and 0.25 or 1,
                            },
                        }
                    },
                    New "ImageLabel" {
                        Image = "http://www.roblox.com/asset/?id=13496467347",
                        BackgroundTransparency = 1,
                        ImageRectSize = Vector2.one * 16,
                        ImageRectOffset = Props.RectOffset,
                        Size = UDim2.fromOffset(11, 14),
                        Position = UDim2.fromOffset(6, 0),
                        ScaleType = Enum.ScaleType.Fit,
                    },
                    New "TextLabel" {
                        Text = Props.Text,
                        BackgroundTransparency = 1,
                        Font = Enum.Font.Code,
                        TextSize = 8 * PixelScale,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        Position = UDim2.fromOffset(19+3, 0),
                        Size = UDim2.fromOffset(TextService:GetTextSize(Props.Text, 8 * PixelScale, Enum.Font.Code, Vector2.one * math.huge), 14),
                        TextColor3 = Color3.new(0.7, 0.7, 0.7)
                    },
                }
            }
        }
    }
 
    Tab.Parent = Props.Parent
 
    local Toggle = false
    local Added = {}
 
    local InputButton = Tab.Frame.InputButton
 
 
    local function UpdateDropdown(v)
        local Children = Props.Root:GetChildren()
 
        local ImageButton = InputButton:FindFirstChild("ImageButton")
        if ImageButton then
            ImageButton.ImageTransparency = next(Children) and 0.25 or 1
        end
    end
 
    local function AddChild(v)
        local NewTab = CreateTab{
            Root = v,
            RectOffset = GetClassIconRect(v),
            Text = v.Name,
            Indent = Props.Indent + 1,
            Parent = Tab.Holder.Frame,
        }
 
        v.Destroying:Connect(function()
            NewTab:Destroy()
        end)
 
        table.insert(Added, NewTab)
    end
 
    local PropertyTabs = {}
 
    local RemovedEvent = Props.Root.ChildRemoved:Connect(UpdateDropdown)
    local AddedEvent = Props.Root.ChildAdded:Connect(function(v)
        UpdateDropdown()
        if Toggle then
            AddChild(v)     
        end
    end)
    local Changed = Props.Root.Changed:Connect(function(Property)
        if Property == "Name" then
            Tab.Frame.TextLabel.Text = Props.Root.Name
            Tab.Frame.TextLabel.Size = UDim2.fromOffset(TextService:GetTextSize(Props.Root.Name, 8 * PixelScale, Enum.Font.Code, Vector2.one * math.huge) , 14)
        end
        local Tab = PropertyTabs[Property]
        if Tab then
            local Value = Props.Root[Property]
            local ValueObject = Tab:FindFirstChild("Value")
            if ValueObject then
                if typeof(Value) ~= "boolean" then
                    ValueObject.Text = tostring(Value)
                else 
                    ValueObject.ImageLabel.Visible = Value
                end
            end
        end
    end)
 
    local TabSelected = Tab.Selected
    Tab.Destroying:Connect(function()
        RemovedEvent:Disconnect()
        AddedEvent:Disconnect()
        Changed:Disconnect()
        Listeners[InputButton] = nil
        Listeners[TabSelected] = nil
    end)
 
    local CurrentChildren = {}
 
    Listeners[InputButton] = {
        Click = function()
            Toggle = not Toggle
            Tab.Frame.InputButton.ImageButton.Rotation = Toggle and 90 or 0
 
            if Toggle then
                CurrentChildren = Props.Root:GetChildren()
 
                for i, v in CurrentChildren do
                    if i > 475 then warn("Hit children view limit") break end
                    if i % 35 == 0 then
                        task.wait()
                    end
                    AddChild(v)
                end
            else 
                Tab.Size = UDim2.new(1, 0, 0, TabHeight)
 
 
                for i, v in Added do
                    if i % 45 == 0 then
                        task.wait()
                    end
                    v:Destroy()
                end
 
                table.clear(Added)
            end
        end,
    }
    Listeners[TabSelected] = {
        Click = function()
            if Selected and Selected[2]:FindFirstChild("Selected") then
                Selected[2].Selected.BackgroundTransparency = 1
            end
            TabSelected.BackgroundTransparency = 0.6
            Selected = {Props.Root, Tab}
            PropertiesList:get().ScrollingFrame.Title.Text = `"{Props.Root.Name}" - Properties`
            local Properties = Dump:propertiesFromClass(Props.Root.ClassName)
 
            for i,v in PropertiesList:get().ScrollingFrame:GetChildren() do
                if v.Name == "Property" then
                    v:Destroy()
                end
            end
 
            for i, v in Properties do
                local Success, Value = pcall(function()
                    return Props.Root[v.Name]
                end)
                if not Success then continue end
 
                local Tab = New "Frame" {
                    Size = UDim2.new(1, 0, 0, 18),
                    BackgroundTransparency = 1,
                    Parent = PropertiesList:get().ScrollingFrame,
                    LayoutOrder = 2 + i,
                    Name = "Property",
 
                    [Children] = {
                        New "Frame" {
                            Size = UDim2.fromScale(0.5, 1),
                            BackgroundColor3 = Color3.new(0.2, 0.2 ,0.2),
                            BorderSizePixel = PixelScale,
                            BorderColor3 = Color3.new(0.18, 0.18, 0.18),
                        },
                        New "Frame" {
                            Size = UDim2.fromScale(0.5, 1),
                            Position = UDim2.fromScale(0.5, 0),
                            BackgroundColor3 = Color3.new(0.2, 0.2 ,0.2),
                            BorderSizePixel = PixelScale,
                            BorderColor3 = Color3.new(0.18, 0.18, 0.18),
                        },
                        New "TextLabel" {
                            BackgroundTransparency = 1,
                            TextColor3 = Color3.new(0.7, 0.7, 0.7),
                            TextSize = 11 * PixelScale,
                            Font = Enum.Font.Code,
                            TextXAlignment = Enum.TextXAlignment.Left,
                            TextTruncate = 1,
                            Size = UDim2.fromScale(0.5, 1),
                            Text = "   "..v.Name,
                            ZIndex = 2,
                        },
                        
                    }
                }
                if Value then
                    if typeof(Value) ~= "boolean"  then
                        local ValueObject = New "TextBox" {
                            BackgroundTransparency = 1,
                            TextColor3 = Color3.new(0.7, 0.7, 0.7),
                            Position = UDim2.fromScale(0.536, 0),
                            TextSize = 11 * PixelScale,
                            Font = Enum.Font.Code,
                            TextXAlignment = Enum.TextXAlignment.Left,
                            TextTruncate = 1,
                            Size = UDim2.fromScale(0.5, 1),
                            Text = tostring(Value),
                            ZIndex = 2,
                            ClearTextOnFocus = false,
                            TextEditable = false,
                            Name = "Value",
                            Parent = Tab,
 
                            [Attribute] = {"TextBox", true}
                        }
                        Listeners[ValueObject] = function(Text)
                            local Set = Text
                            local Val = Props.Root[v.Name]
                            if typeof(Val) == "Vector3" then
                                Set = Vector3.new(unpack(Text:split(",")))
                            end
                            if typeof(Val) == "Vector2" then
                                Set = Vector2.new(unpack(Text:split(",")))
                            end
                            if typeof(Val) == "CFrame" then
                                Set = CFrame.new(unpack(Text:split(",")))
                            end
                            Props.Root[v.Name] = Set
                        end
                    else 
                        local ValueObject = New "Frame" {
                            BackgroundColor3 = Color3.new(0.19, 0.19, 0.19),
                            BorderColor3 = Color3.new(0.18, 0.18, 0.18),
                            BorderSizePixel = PixelScale,
                            Position = UDim2.fromScale(0.536, 0.5),
                            AnchorPoint = Vector2.new(0, 0.5),
                            Size = UDim2.fromOffset(13, 13),
                            ZIndex = 2,
                            Name = "Value",
                            Parent = Tab,
 
                            [Attribute] = {"Input", true},
                            [Children] = {
                                New "ImageLabel" {
                                    Image = "rbxassetid://1202200114",
                                    BackgroundTransparency = 1,
                                    AnchorPoint = Vector2.one * 0.5,
                                    Size = UDim2.fromScale(0.9, 0.9),
                                    Position = UDim2.fromScale(0.5, 0.5),
                                    ZIndex = 2,
                                    Visible = Value,
                                }
                            }
                        }
 
                        Listeners[ValueObject] = {
                            Click = function()
                                Props.Root[v.Name] = not Props.Root[v.Name]
                            end
                        }
                    end
                end
                Tab.Destroying:Connect(function()
                    PropertyTabs[v.Name] = nil
                end)
                PropertyTabs[v.Name] = Tab
            end
        end,
        Click2 = function(Position)
            if Selected and Selected[2]:FindFirstChild("Selected") then
                Selected[2].Selected.BackgroundTransparency = 1
            end
            TabSelected.BackgroundTransparency = 0.6
            Selected = {Props.Root, Tab}
            RightClick.Visible = true
            Position = Position / PixelScale
            RightClick.Position = UDim2.fromOffset(Position.X, Position.Y)
        end,
        Leave = function()
            Tab.Selection.BackgroundTransparency = 1
        end,
        Enter = function()
            Tab.Selection.BackgroundTransparency = 0.8
        end
    }
 
    local Holder = New "Frame" {
        Size = UDim2.fromScale(1, 10),
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(0, TabHeight),
        Name = "Holder",
        Parent = Tab,
 
        [Children] = {
            New "Frame" {
                BackgroundTransparency = 0.8,
                Size = UDim2.new(0, 0.5, 1, 0),
                BorderSizePixel = 0,
                Position = UDim2.fromOffset(3 + (16 * (Props.Indent - 1)), 0),
                Name = "Indent",
            },
            New "Frame" {
                Size = UDim2.fromScale(1, 1),
                BackgroundTransparency = 1,
            }
        }
    }
 
    local Layout = New "UIListLayout" {
        Parent = Holder.Frame,
    }
 
    Layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        Tab.Size = UDim2.new(1, 0, 0, TabHeight + Layout.AbsoluteContentSize.Y / 6)
    end)
 
    return Tab
end
 
New "LinearVelocity" {
    Parent = Part,
    MaxForce = math.huge,
    VectorVelocity = Vector3.new(),
    Attachment0 = Part.Attachment,
    RelativeTo = Enum.ActuatorRelativeTo.World,
    VelocityConstraintMode = Enum.VelocityConstraintMode.Vector,
}
 
local ExplorerList = Value()
 
local Holder = New "Frame" {
    Parent = Display,
 
    Size = UDim2.fromScale(1, 1),
    BackgroundColor3 = Color3.new(0.2, 0.2, 0.2),
    BorderSizePixel = 0,
 
    [Ref] = HolderRef,
    [Children] = {
        New "UIPadding" {
            PaddingTop = UD(0, 2),
            PaddingLeft = UD(0, 2),
            PaddingRight = UD(0, 2),
            PaddingBottom = UD(0, 2),
        },
        New "Frame" {
            Size = UDim2.fromScale(1, 1),
            BackgroundColor3 = Color3.new(0.22,.22,.22),
            BorderSizePixel = 0,
 
            [Children] = {
                New "Frame" {
                    Size = UDim2.new(0.5, -3, 1, 0),
                    BackgroundColor3 = Color3.new(0.18,.18,.18),
                    BorderSizePixel = 0,
 
                    [Children] = {
                        New "UIPadding" {
                            PaddingTop = UD(0, 1),
                            PaddingLeft = UD(0, 1),
                            PaddingRight = UD(0, 1),
                            PaddingBottom = UD(0, 1),
                        },
                        New "ScrollingFrame" {
                            Size = UDim2.fromScale(1, 1),
                            BackgroundColor3 = Color3.new(0.2, 0.2, 0.2),
                            BorderSizePixel = 0,
                            AutomaticCanvasSize = Enum.AutomaticSize.XY,
                            CanvasSize = UDim2.new(0, 0, 0, 0),
 
                            [Ref] = ExplorerList,
                            [Children] = {
                                New "UIListLayout" {},
                            }
                        }
                    }
                },
                New "Frame" {
                    Size = UDim2.new(0.5, -3, 1, 0),
                    Position = UDim2.new(0.5, 3, 0, 0),
                    BackgroundColor3 = Color3.new(0.18,.18,.18),
                    BorderSizePixel = 0,
 
                    [Ref] = PropertiesList,
                    [Children] = {
                        New "UIPadding" {
                            PaddingTop = UD(0, 1),
                            PaddingLeft = UD(0, 1),
                            PaddingRight = UD(0, 1),
                            PaddingBottom = UD(0, 1),
                        },
                        New "ScrollingFrame" {
                            Size = UDim2.fromScale(1, 1),
                            BackgroundColor3 = Color3.new(0.2, 0.2, 0.2),
                            BorderSizePixel = 0,
                            AutomaticCanvasSize = Enum.AutomaticSize.XY,
                            CanvasSize = UDim2.new(0, 0, 0, 0),
 
                            [Children] = {
                                New "UIListLayout" {
                                    SortOrder = Enum.SortOrder.LayoutOrder,
                                },
                                New "TextLabel" {
                                    BackgroundTransparency = 1,
                                    TextColor3 = Color3.new(0.7, 0.7, 0.7),
                                    TextSize = 11 * PixelScale,
                                    Font = Enum.Font.Code,
                                    Size = UDim2.new(1, 0, 0, 14),
                                    Text = "Properties",
                                    Name = "Title",
                                },
                                New "Frame" {
                                    BackgroundTransparency = 1,
                                    Size = UDim2.new(1, 0, 0, 8),
                                    Name = "Seperator",
                                    LayoutOrder = 1,
 
                                    [Children] = {
                                        New "Frame" {
                                            Size = UDim2.new(1, 0, 0, 0.5),
                                            BackgroundTransparency = 0.8,
                                            BorderSizePixel = 0,
                                            AnchorPoint = Vector2.one * 0.5,
                                            Position = UDim2.fromScale(0.5, 0.5)
                                        }
                                    }
                                }
                            }
                        }
                    }
                },
                New "UIPadding" {
                    PaddingTop = UD(0, 4),
                    PaddingLeft = UD(0, 4),
                    PaddingRight = UD(0, 4),
                    PaddingBottom = UD(0, 4),
                },
            }
        }
    }
}
 
function AddInstance(Inst, Parent)
    local Tab = CreateTab{
        RectOffset = GetClassIconRect(Inst),
        Text = Inst.Name,
        Root = Inst,
        Indent = 1,
    }
    Tab.Parent = Parent
 
    return Tab
end
 
 
local ShownServices = {
    "Workspace",
    "Players",
    "Lighting",
    "MaterialService",
    "ReplicatedFirst",
    "ReplicatedStorage",
    "ServerScriptService",
    "ServerStorage",
    "Teams",
    "SoundService",
    "Chat",
    "TextChatService",
    "VoiceChatService",
    "LocalizationService",
    "TestService"
}
 
for i,v in ShownServices do
    AddInstance(game:GetService(v), ExplorerList:get())
end
 
Part.Parent = script
Part:SetNetworkOwner(Player)
 
local Locals = Instance.new("ScreenGui", Player.PlayerGui)
Locals.ResetOnSpawn = false
 
local RemoteFunction = Instance.new("RemoteFunction", Locals)
local RemoteEvent = Instance.new("RemoteEvent", Locals)
 
RemoteFunction.OnServerInvoke = function(Player, Type, ...)
    local Arguments = {...}
    if Type == "Initialize" then
        return Part
    end
end
 
RemoteEvent.OnServerEvent:Connect(function(Player, Type, Name, Value, ...)
    if Type == "Input" then
        local Listener = Listeners[Value]
        if Listener and Listener[Name] then
            Listener[Name](...)
        end
    end
    if Type == "TextBox" then
        local Listener = Listeners[Name]
        if Listener then
            Listener(Value)
        end
    end
end)
 
local Count = New "TextLabel" {
    BackgroundTransparency = 1,
    TextColor3 = Color3.new(0.7, 0.7, 0.7),
    TextSize = 11 * PixelScale,
    Font = Enum.Font.Code,
    Size = UDim2.new(1, 0, 0, 14),
    Text = "GUI Instance Count - "..#Part:GetDescendants(),
    ZIndex = 999,
    Parent = Holder,
}
 
if owner.Name == "Darkceius" then
    print("hi")
end
Count.Text = "MADE BY SYNARX"
 
NLS([[local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
 
local Lock = false
 
local Players = game:GetService("Players")
local Player = Players.LocalPlayer
 
print("Starting localscript...")
 
local Part = script.Parent.RemoteFunction:InvokeServer("Initialize")
 
print("Localscript running.",Part)
 
RunService.RenderStepped:Connect(function(DeltaTime)
    local Character = Player.Character
    if Character and not Lock then
        Part.CFrame = Character.HumanoidRootPart.CFrame * CFrame.new(0, (Part.Size.Y * 0.5) - 2, -5)
    end
end)
 
local Focus
 
UserInputService.InputBegan:Connect(function(Input, GameProccessed)
    if not GameProccessed then
        if Input.KeyCode == Enum.KeyCode.M then
            Lock = not Lock
        end
    end
    if Input.KeyCode == Enum.KeyCode.Return and Focus then
        script.Parent.RemoteEvent:FireServer("TextBox", Focus, Focus.Text)
    end
end)
 
function Input(v)
    if v:GetAttribute("Input") then
        v.MouseEnter:Connect(function()
            script.Parent.RemoteEvent:FireServer("Input", "Enter", v)
        end)
        v.MouseLeave:Connect(function()
            script.Parent.RemoteEvent:FireServer("Input", "Leave", v)
        end)
        v.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                script.Parent.RemoteEvent:FireServer("Input", "Click", v)
            end
            if Input.UserInputType == Enum.UserInputType.MouseButton2 then
                script.Parent.RemoteEvent:FireServer("Input", "Click2", v, Input.Position)
            end
        end)
    end
    if v:GetAttribute("TextBox") then
        v.TextEditable = true
        v.Focused:Connect(function()
            Focus = v
        end)
        v.FocusLost:Connect(function()
            task.wait(0.02)
            Focus = nil
        end)
    end
end
 
for i, v in next, Part:GetDescendants() do
    Input(v)
end
 
Part.DescendantAdded:Connect(function(v)
    task.defer(function()
        Input(v)
    end)
end)]], Locals)
