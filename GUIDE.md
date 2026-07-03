# rsg-menubase User Guide

## Overview

rsg-menubase is a menu framework for RedM that provides a simple API for creating styled, navigable menus. Other resources call its exports to open menus with custom elements.

---

## Installation

1. Download/place the `rsg-menubase` resource in your server's resources folder
2. Add `ensure rsg-menubase` to your `server.cfg` (before resources that use it)
3. Ensure you have `rsg-core` installed as a dependency

**Requirements:**
- RedM server (rdr3)
- rsg-core framework

---

## Configuration

The menu has no config file - all configuration is done through the API when opening menus. Key options passed via data parameter:

| Option | Type | Description |
|--------|------|-------------|
| `title` | string | Menu header title |
| `subtext` | string | Subtitle/description text |
| `elements` | table | Array of menu items |
| `disableMovement` | boolean | Freeze player while menu open |
| `lockInventory` | boolean | Block inventory access while open |
| `isGrid` | boolean | Render as grid layout |

---

## Usage

### Opening a Menu

```lua
local menu = MenuData.Open(type, namespace, name, data, submit, cancel, change, close)
```

**Parameters:**
- `type` - Menu type (default: `"default"`)
- `namespace` - Unique namespace for your resource
- `name` - Unique menu name within namespace
- `data` - Table containing title, elements, options
- `submit` - Function called when item selected
- `cancel` - Function called when menu closed without selection
- `change` - Function called on navigation change
- `close` - Function called when menu closes

### Example Menu

```lua
local menuData = {
    title = "Shop Menu",
    subtext = "Select an item to purchase",
    elements = {
        { label = "Bread", value = "bread", desc = "Fresh baked bread - $5" },
        { label = "Water", value = "water", desc = "Clean water - $2" },
        { label = "Coffee", value = "coffee", desc = "Hot coffee - $3" }
    }
}

local menu = MenuData.Open("default", "myshop", "main", menuData, 
    function(data, menu) -- submit
        print("Selected: " .. data.current.value)
        menu.close()
    end,
    function(data, menu) -- cancel
        print("Menu cancelled")
    end,
    function(data, menu) -- change
        print("Navigation changed")
    end,
    function() -- close
        print("Menu closed")
    end
)
```

### Element Types

**Default Item:**
```lua
{ label = "Item Name", value = "item_value", desc = "Description text" }
```

**Slider Item:**
```lua
{ 
    label = "Quantity", 
    type = "slider", 
    value = 1, 
    min = 1, 
    max = 10, 
    hop = 1,
    desc = "Select quantity"
}
```

**Slider with Options:**
```lua
{
    label = "Color",
    type = "slider",
    value = 1,
    options = { "Red", "Green", "Blue" }
}
```

**Grid Item (with image):**
```lua
{ label = "Item", value = "item", image = "item_image" }
```

### Menu Methods

| Method | Description |
|--------|-------------|
| `menu.close()` | Close the menu |
| `menu.refresh()` | Re-render menu with updated data |
| `menu.update(query, newData)` | Update elements matching query |
| `menu.setElement(i, key, val)` | Set specific element property |
| `menu.setElements(newElements)` | Replace all elements |
| `menu.setTitle(title)` | Change menu title |
| `menu.addNewElement(element)` | Add item to menu |
| `menu.removeElementByValue(value)` | Remove item by value |

### Utility Functions

```lua
-- Close specific menu
MenuData.Close("default", "namespace", "name")

-- Close all open menus
MenuData.CloseAll()

-- Check if menu is open
if MenuData.IsOpen("default", "namespace", "name") then
    print("Menu is open")
end

-- Get opened menu reference
local openedMenu = MenuData.GetOpened("default", "namespace", "name")

-- Get all opened menus
local allMenus = MenuData.GetOpenedMenus()
```

---

## Controls

| Key | Action |
|-----|--------|
| ENTER | Select item |
| BACKSPACE | Close menu / Go back |
| UP ARROW | Navigate up |
| DOWN ARROW | Navigate down |
| LEFT ARROW | Slider decrease |
| RIGHT ARROW | Slider increase |

---

## Communication Flow

```
┌─────────────┐    SendNUIMessage     ┌─────────────┐
│   Client    │ ───────────────────── │     NUI     │
│  (Lua)      │                       │  (JS/HTML)  │
│             │ ◄───────────────────── │             │
└─────────────┘   RegisterNUICallback  └─────────────┘

Client → NUI:
  - openMenu: Open with data
  - closeMenu: Close menu
  - controlPressed: Forward key input

NUI → Client:
  - menu_submit: Item selected
  - menu_cancel: Menu cancelled
  - menu_change: Selection changed
  - playsound: Play navigation sound
  - update_last_selected: Remember position
```

---

## Troubleshooting

### Menu Not Appearing
- Ensure `rsg-menubase` is started before resources using it
- Check server console for errors
- Verify `rsg-core` is running

### Controls Not Working
- Menu must have focus (no other NUI open)
- Check if another resource is blocking controls

### Menu Doesn't Close
- Use `MenuData.CloseAll()` to force close all menus
- Check for errors in client console (`F8`)

### Slider Values Not Updating
- Ensure slider has `type = "slider"` set
- Verify `min`, `max`, and `value` are numbers

### Last Selection Not Remembered
- Selection persistence only works within same session
- Cleared on resource restart

---

## Server Integration

The current server-side implementation is minimal. To add persistent settings:

```lua
-- server/server.lua
local RSGCore = exports['rsg-core']:GetCoreObject()

RegisterNetEvent('rsg-menubase:server:saveSettings', function(data)
    local src = source
    local Player = RSGCore.Functions.GetPlayer(src)
    if not Player then return end
    
    local citizenid = Player.PlayerData.citizenid
    -- Save to database or file
end)

RegisterNetEvent('rsg-menubase:server:loadSettings', function()
    local src = source
    local Player = RSGCore.Functions.GetPlayer(src)
    if not Player then return end
    
    -- Load and return settings
    TriggerClientEvent('rsg-menubase:client:settingsLoaded', src, settings)
end)
```

---

## Export

Get the MenuData object from any resource:

```lua
local MenuData = exports['rsg-menubase']:GetMenuData()
```

Or use the event:

```lua
TriggerEvent('rsg-menubase:getData', function(menuData)
    -- Use menuData
end)
```
