# Michs_PixelPerfectLib

Library to help with pixel perfect sizes while creating and positioning elements

## How Scaling Works

The library uses 1080p as its baseline and automatically scales dimensions based on the game's display resolution

For example, using `ToUI(100)` with the default global scale of 100%:

| Game Resolution | Physical Size |
|---|---|
| 1920×1080 | 100 pixels |
| 2560×1440 | 133 pixels |
| 3840×2160 | 200 pixels |

Dimensions are rounded to whole physical pixels and converted into WoW UI units, accounting for the game's UI scale.

The global scale can also be adjusted between 50% and 200% using `SetGlobalScale()`.

## Installation

Michs_PixelPerfectLib can be installed in one of two ways:

- Standalone dependency: users install Michs_PixelPerfectLib separately.
- Embedded library: you ship the library files inside your addon.

### Standalone Dependency

Use this setup when you want users to install [Michs_PixelPerfectLib from CurseForge](https://www.curseforge.com/wow/addons/michs-pixelperfectlib) separately.

Add it as a required dependency in your addon's `.toc` file:

```toc
## Dependencies: Michs_PixelPerfectLib
```

On CurseForge, mark `Michs_PixelPerfectLib` as a required dependency.

### Embedded Library

Copy the `Libs` folder into your addon and load LibStub before Michs_PixelPerfectLib in your addon's `.toc` file:

```toc
Libs/LibStub/LibStub.lua
Libs/MichsPixelPerfectLib-2.0/MichsPixelPerfectLib-2.0.lua
```

The library files must be loaded before any addon file that calls `LibStub("MichsPixelPerfectLib-2.0")`.

Do not add `Michs_PixelPerfectLib` as a dependency when embedding the library files directly.

## Quick Start

Create a scaler once and reuse it throughout your addon.

```lua
local _, addon = ...

addon.PixelPerfect = LibStub("MichsPixelPerfectLib-2.0"):CreateScaler()
```

Use `ToUI` for dimensions and offsets, and `CenterElement` for centered positioning.

```lua
local _, addon = ...

local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent, "BackdropTemplate")

frame:SetBackdrop({ bgFile = "Interface\\Buttons\\WHITE8X8" })
frame:SetBackdropColor(0.2, 0.2, 0.2, 1)

PP:RegisterForUpdate(function()
    frame:SetSize(PP:ToUI(240), PP:ToUI(120))
    PP:CenterElement(frame, UIParent, 0, 0)
end)
```

**Important:**
- `ToUI(value)` converts a value into resolution-scaled, pixel-snapped UI units.
- `CenterElement(element, parent, offsetX, offsetY)` takes **raw offsets**. Do not apply `ToUI` to them.
- `RegisterForUpdate(callback)` runs immediately when registered and again whenever the global scale, UI scale, or screen resolution changes.
- Create the scaler before any files that use it.

For additional configuration and examples, see the sections below.

## API Reference

| Method | Description |
|---|---|
| `CreateScaler()` | Creates a new scaler instance. |
| `SetGlobalScale(scale)` | Sets the global scale (0.5–2.0, default 1.0). |
| `ToUI(value)` | Converts a value to resolution-scaled, pixel-snapped UI units. |
| `CenterElement(element, parent, x, y)` | Centers an element using raw offsets. |
| `RegisterForUpdate(callback)` | Runs a callback immediately and on scale or resolution changes. |

## How To Use


### Create Scaler

As shown in Quick Start, create one scaler for your addon and reuse it everywhere to keep the global scale consistent.

Access the shared scaler from other files:

```lua
local _, addon = ...

local PP = addon.PixelPerfect
```

<details>
<summary><strong>Initialization timing</strong></summary>

The scaler must be created before any code uses it.

If the file that creates your scaler loads first, other files can access `addon.PixelPerfect` directly.

Otherwise, access the scaler inside an initialization function after it has been created.

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

addon.PixelPerfect = LibStub("MichsPixelPerfectLib-2.0"):CreateScaler()

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

Use `ToUI` for sizes, font sizes, and `SetPoint` offsets. Pass raw offsets to `CenterElement`, which scales them internally.

<details>
<summary><strong>ToUI</strong></summary>

`ToUI(value)` converts the value into resolution-scaled, pixel-snapped UI units.

Sizes and offsets scale with the game's display height, using 1080p as the baseline:

```text
value -> resolution scale -> global scale -> round to whole pixels -> UI units
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

</details>

<details>
<summary><strong>Font Size</strong></summary>

Use `ToUI(value)` for font sizes, just as for other dimensions.

Basic font sizing example:

```lua
local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent)
local fontString = frame:CreateFontString(nil, "OVERLAY")
local fontSize = 20

fontString:SetFont("Fonts\\FRIZQT__.TTF", PP:ToUI(fontSize), "OUTLINE")
```

</details>

<details>
<summary><strong>CenterElement</strong></summary>

`CenterElement(element, parent, offsetX, offsetY)` centers an element inside a parent.

Avoid using `SetPoint("CENTER", parent, "CENTER", 0, 0)`.<br>
`SetPoint` is fine to use for non-centered anchors.

Pass raw offsets into `CenterElement`. It applies resolution and global scaling internally; do not convert these offsets with `ToUI` first.

Basic `CenterElement` usage:

```lua
local PP = addon.PixelPerfect

local frame = CreateFrame("Frame", nil, UIParent)
local width = 240
local height = 120
local offsetX = 0
local offsetY = 0

frame:SetSize(PP:ToUI(width), PP:ToUI(height))
PP:CenterElement(
	frame,
	UIParent,
	offsetX,
	offsetY
)
```

</details>

### Updating Layout

Register an update callback after creating your elements.<br>
The library will run this callback when your layout should be recalculated.

The updates will be executed when:

- The callback is registered.
- Your addon changes global scale through `SetGlobalScale`.
- The game UI scale or game display resolution changes.

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
	frame:SetSize(PP:ToUI(width), PP:ToUI(height))
	PP:CenterElement(
		frame,
		UIParent,
		offsetX,
		offsetY
	)
	fontString:SetFont("Fonts\\FRIZQT__.TTF", PP:ToUI(fontSize), "OUTLINE")
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
	MainPanel:SetSize(PP:ToUI(240), PP:ToUI(120))
	PP:CenterElement(MainPanel, UIParent, 0, 0)
end

local function UpdateTitleLayout()
	MainPanel.titleText:SetFont("Fonts\\FRIZQT__.TTF", PP:ToUI(14), "OUTLINE")
	MainPanel.titleText:ClearAllPoints()
	MainPanel.titleText:SetPoint(
		"TOPLEFT",
		MainPanel,
		"TOPLEFT",
		PP:ToUI(12),
		PP:ToUI(-12)
	)
end

local function UpdateCloseButtonLayout()
	MainPanel.closeButton:SetSize(PP:ToUI(24), PP:ToUI(24))
	MainPanel.closeButton:ClearAllPoints()
	MainPanel.closeButton:SetPoint(
		"TOPRIGHT",
		MainPanel,
		"TOPRIGHT",
		PP:ToUI(-12),
		PP:ToUI(-12)
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
