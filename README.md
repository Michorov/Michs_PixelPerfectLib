# Michs_PixelPerfectLib

Library to help with pixel perfect sizes and borders while creating and positioning elements

<br>

## Installation

Michs_PixelPerfectLib can be installed in one of two ways:

- Standalone dependency: users install Michs_PixelPerfectLib separately.
- Embedded library: you ship the library files inside your addon.

### Standalone Dependency

Use this setup when you want users to install `Michs_PixelPerfectLib` separately from CurseForge.

Then add it as a required dependency in your addon's `.toc` file:

```toc
## Dependencies: Michs_PixelPerfectLib
```

On CurseForge, mark `Michs_PixelPerfectLib` as a required dependency.

### Embedded Library

Copy the `Libs` folder into your addon and load LibStub before Michs_PixelPerfectLib in your addon's `.toc` file:

```toc
Libs/LibStub/LibStub.lua
Libs/MichsPixelPerfectLib-1.0/MichsPixelPerfectLib-1.0.lua
```

The library files must be loaded before any addon file that calls `LibStub("MichsPixelPerfectLib-1.0")`.

Do not add `Michs_PixelPerfectLib` as a dependency when embedding the library files directly.

<br>

## How To Use

### Create Scaler

Create one scaler for your addon and reuse it everywhere.<br>
This keeps your global scale setting consistent across all files.

Example:

```lua
local _, addon = ...

addon.PixelPerfect = LibStub("MichsPixelPerfectLib-1.0"):CreateScaler()
```

Then use the same scaler from other files:

```lua
local _, addon = ...

local PP = addon.PixelPerfect
```

<details>
<summary><strong>Initialization timing</strong></summary>

The scaler must be created before any code uses it.

If the file that creates your scaler loads before the rest of your addon files, later files can use `addon.PixelPerfect` directly.

If the file that creates your scaler loads after the rest of your addon files, those files should only define tables and functions at load time. Access `addon.PixelPerfect` inside an initialization function after the scaler is created.

```lua
-- MainPanel.lua
local _, addon = ...

addon.MainPanel = {}

function addon.MainPanel:Initialize()
	local PP = addon.PixelPerfect

	-- Use PP here.
end
```

```lua
-- Core.lua
local _, addon = ...

addon.PixelPerfect = LibStub("MichsPixelPerfectLib-1.0"):CreateScaler()

addon.MainPanel:Initialize()
```

</details>

### Set Global Scale

The library uses a default global scale of 100%.

Global scale is clamped between 50% and 200%. This means `0.5` is 50%, `1` is 100%, and `2` is 200%.

You can change the global scale through the scaler API:

```lua
local _, addon = ...

addon.PixelPerfect:SetGlobalScale(1.25)
```

### Sizing And Positioning

Every size or offset value should go through the scaling API before it is applied to an element.

<details>
<summary><strong>ToUI & ToUIScaled</strong></summary>

`ToUI(value)` converts the value into pixel-snapped UI units.

Use `ToUI` when the value should stay the same pixel count on every resolution:

```text
value -> global scale -> UI units
```

Basic `ToUI` usage:

```lua
local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent)
local width = 240
local height = 120
local offsetX = 16
local offsetY = -16

frame:SetSize(PP:ToUI(width), PP:ToUI(height))
frame:SetPoint(
	"TOPLEFT",
	UIParent,
	"TOPLEFT",
	PP:ToUI(offsetX),
	PP:ToUI(offsetY)
)
```

`ToUIScaled(value)` converts the value into screen resolution scaled pixel-snapped UI units.

Use `ToUIScaled` when the value should scale up or down with the user's resolution:

```text
value -> resolution scale -> global scale -> UI units
```

Basic `ToUIScaled` usage:

```lua
local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent)
local width = 240
local height = 120
local offsetX = 16
local offsetY = -16

frame:SetSize(PP:ToUIScaled(width), PP:ToUIScaled(height))
frame:SetPoint(
	"TOPLEFT",
	UIParent,
	"TOPLEFT",
	PP:ToUIScaled(offsetX),
	PP:ToUIScaled(offsetY)
)
```

</details>

<details>
<summary><strong>ScaleFont</strong></summary>

`ScaleFont(value)` scales a font size using the same global scale.

Basic `ScaleFont` usage:

```lua
local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent)
local fontString = frame:CreateFontString(nil, "OVERLAY")
local fontSize = 20

fontString:SetFont("Fonts\\FRIZQT__.TTF", PP:ScaleFont(fontSize), "OUTLINE")
```

</details>

<details>
<summary><strong>CenterElement</strong></summary>

`CenterElement(element, parent, offsetX, offsetY)` centers an element inside a parent.

Avoid using `SetPoint("CENTER", parent, "CENTER", 0, 0)`.<br>
`SetPoint` is fine to use for non-centered anchors.

The offsets passed into `CenterElement` must already go through `ToUI` or `ToUIScaled`.

Basic `CenterElement` usage:

```lua
local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent)
local width = 240
local height = 120
local offsetX = 0
local offsetY = 0

frame:SetSize(PP:ToUIScaled(width), PP:ToUIScaled(height))
PP:CenterElement(
	frame,
	UIParent,
	PP:ToUIScaled(offsetX),
	PP:ToUIScaled(offsetY)
)
```

</details>

### Updating Layout

Register an update callback after creating your elements.<br>
The library will run this callback when your layout should be recalculated.

The Updates will be executed when:

- The callback is registered.
- Your addon changes global scale through `SetGlobalScale`.
- The game UI scale or monitor resolution changes.

Basic `RegisterForUpdate` example:

```lua
local _, addon = ...

local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent)
local fontString = frame:CreateFontString(nil, "OVERLAY")
fontString:SetText("Pixel Perfect")

local width = 240
local height = 120
local offsetX = 0
local offsetY = 0
local fontSize = 20

local function UpdateLayout()
	frame:SetSize(PP:ToUIScaled(width), PP:ToUIScaled(height))
	PP:CenterElement(
		frame,
		UIParent,
		PP:ToUIScaled(offsetX),
		PP:ToUIScaled(offsetY)
	)
	fontString:SetFont("Fonts\\FRIZQT__.TTF", PP:ScaleFont(fontSize), "OUTLINE")
end

PP:RegisterForUpdate(function()
	UpdateLayout()
end)
```

The update callback receives an optional `updateEvent` argument:

```lua
PP:RegisterForUpdate(function(updateEvent)
	UpdateLayout()
end)
```

`updateEvent` is the event that triggered the layout update.

You can ignore `updateEvent` if your layout update does not need it.

The callback must be registered after the elements are created.

General updating guidelines:

- You can register as many update callbacks as your addon needs.
- Update callbacks are executed in the order they are registered.
- Update size before position within each element.
- Update parent elements before child elements.

<br>

## Examples

### Full Element Implementation Example

```lua
local _, addon = ...

local PP = addon.PixelPerfect

local MainPanel = CreateFrame("Frame", nil, UIParent)

MainPanel.titleText = MainPanel:CreateFontString(nil, "OVERLAY")
MainPanel.titleText:SetText("Pixel Perfect")

MainPanel.closeButton = CreateFrame("Button", nil, MainPanel, "UIPanelButtonTemplate")

local function UpdateMainPanelLayout()
	MainPanel:SetSize(PP:ToUIScaled(240), PP:ToUIScaled(120))
	PP:CenterElement(MainPanel, UIParent, PP:ToUIScaled(0), PP:ToUIScaled(0))
end

local function UpdateTitleLayout()
	MainPanel.titleText:SetFont("Fonts\\FRIZQT__.TTF", PP:ScaleFont(14), "OUTLINE")
	MainPanel.titleText:ClearAllPoints()
	MainPanel.titleText:SetPoint(
		"TOPLEFT",
		MainPanel,
		"TOPLEFT",
		PP:ToUIScaled(12),
		PP:ToUIScaled(-12)
	)
end

local function UpdateCloseButtonLayout()
	MainPanel.closeButton:SetSize(PP:ToUIScaled(24), PP:ToUIScaled(24))
	MainPanel.closeButton:ClearAllPoints()
	MainPanel.closeButton:SetPoint(
		"TOPRIGHT",
		MainPanel,
		"TOPRIGHT",
		PP:ToUIScaled(-12),
		PP:ToUIScaled(-12)
	)
end

PP:RegisterForUpdate(function()
	UpdateMainPanelLayout()
end)

PP:RegisterForUpdate(function()
	UpdateTitleLayout()
end)

PP:RegisterForUpdate(function()
	UpdateCloseButtonLayout()
end)
```

You can also register one update callback and call multiple layout functions inside it:

```lua
PP:RegisterForUpdate(function()
	UpdateMainPanelLayout()
	UpdateTitleLayout()
	UpdateCloseButtonLayout()
end)
```
