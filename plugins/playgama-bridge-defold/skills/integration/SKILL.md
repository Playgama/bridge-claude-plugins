---
name: integration
description: Use this skill when working with Playgama Bridge SDK for Defold games. Activates for game development involving cross-platform publishing, advertisements, in-app purchases, leaderboards, or social features using Playgama Bridge in Defold/Lua.
---

# Playgama Bridge SDK Integration for Defold

Playgama Bridge is a cross-platform SDK for publishing Defold HTML5 games across 20+ platforms including Playgama, YouTube, Yandex Games, Crazy Games, Poki, Facebook, Telegram, Xiaomi, and more.

## Installation

1. Copy the `Source code.zip` link from the [GitHub releases](https://github.com/playgama/bridge-defold/releases) page
2. Add it as a dependency in your project settings (`game.project` file)
3. Select `Project` -> `Fetch Libraries` to download the SDK

## Configuration

1. Download the `playgama-bridge-config.json` file from the GitHub releases
2. Place it in the `res/web` folder
3. Ensure the `/res` directory is added to `Bundle Resources` in project settings
4. Configure game identifiers and in-app purchases in the config file

## HTML5 Export

When building for HTML5, select the appropriate index template provided by the SDK to ensure proper initialization.

## Initialization

Once the game is loaded, the plugin is already initialized. No additional actions are required. Access all modules through the `bridge` module:

```lua
local bridge = require("bridge.bridge")

bridge.platform
bridge.player
bridge.device
bridge.storage
bridge.advertisement
bridge.payments
bridge.leaderboards
bridge.social
bridge.achievements
```

After game is fully loaded, send the game ready message:

```lua
bridge.platform.send_message("game_ready")
```

## Device

```lua
local bridge = require("bridge.bridge")

-- Device type: 'mobile', 'tablet', 'desktop', 'tv'
local device_type = bridge.device.type()
```

## Platform

```lua
local bridge = require("bridge.bridge")

-- Platform ID: 'playgama', 'vk', 'yandex', 'crazy_games', etc.
local platform_id = bridge.platform.id()

-- User language (ISO 639-1): 'en', 'ru', etc.
local language = bridge.platform.language()

-- Top-level domain: 'com', 'ru', or nil
local tld = bridge.platform.tld()

-- URL payload (auxiliary information from game URL)
local payload = bridge.platform.payload()
```

### Server Time

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.platform.get_server_time(function (_, data)
        print("Server time: ", data)
    end, function ()
        -- error
    end)
end
```

### Platform Messages

```lua
bridge.platform.send_message("game_ready")
bridge.platform.send_message("gameplay_started")
bridge.platform.send_message("gameplay_stopped")
bridge.platform.send_message("in_game_loading_started")
bridge.platform.send_message("in_game_loading_stopped")
bridge.platform.send_message("player_got_achievement")
```

### Audio State Listener

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.platform.on("audio_state_changed", function (_, is_enabled)
        print("Is audio enabled: ", is_enabled)
        -- Mute/unmute game audio based on is_enabled
    end)
end
```

### Pause State Listener

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.platform.on("pause_state_changed", function (_, is_paused)
        print("Is paused: ", is_paused)
        -- Pause/resume game based on is_paused
    end)
end
```

## Player

```lua
local bridge = require("bridge.bridge")

-- Check authorization support
local is_auth_supported = bridge.player.is_authorization_supported()

-- Check if player is authorized
local is_authorized = bridge.player.is_authorized()

-- Player ID (nil if not authorized)
local player_id = bridge.player.id()

-- Player name (nil if unavailable)
local player_name = bridge.player.name()

-- Player photos (array of URLs sorted by resolution, or empty array)
local photos = bridge.player.photos()

-- Platform-specific player data
local extra = bridge.player.extra()
```

### Authorization

```lua
local bridge = require("bridge.bridge")

function init(self)
    local options = {}

    bridge.player.authorize(
        options,
        function ()
            -- success
            print("Authorized: ", bridge.player.id())
        end,
        function ()
            -- error
            print("Authorization failed")
        end)
end
```

## Storage

### Get Data

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.storage.get(
        { "coins", "level" },
        function (_, data)
            if data.coins then
                print("Coins: ", data.coins)
            end
            if data.level then
                print("Level: ", data.level)
            end
        end,
        function ()
            -- error
        end
    )
end
```

### Set Data

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.storage.set(
        { coins = 42, level = "dungeon" },
        function (_)
            -- success
            print("Data saved")
        end,
        function (_)
            -- error
        end
    )
end
```

### Delete Data

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.storage.delete(
        { "coins", "level" },
        function ()
            -- success
        end,
        function ()
            -- error
        end
    )
end
```

## Banner Ads

```lua
local bridge = require("bridge.bridge")

-- Check support
local is_supported = bridge.advertisement.is_banner_supported()

-- Show banner
function init(self)
    local position = "bottom"  -- 'top' or 'bottom', default = 'bottom'
    local placement = "menu"   -- optional
    bridge.advertisement.show_banner(position, placement)
end

-- Hide banner
bridge.advertisement.hide_banner()
```

### Banner State

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.advertisement.on("banner_state_changed", function (_, state)
        -- States: 'loading', 'shown', 'hidden', 'failed'
        print("Banner state changed: ", state)
    end)
end
```

## Interstitial Ads

```lua
local bridge = require("bridge.bridge")

-- Check support
local is_supported = bridge.advertisement.is_interstitial_supported()

-- Get current state: 'loading', 'opened', 'closed', 'failed'
local state = bridge.advertisement.interstitial_state()

-- Get/set minimum delay between ads (default: 60 seconds)
local delay = bridge.advertisement.minimum_delay_between_interstitial()
bridge.advertisement.set_minimum_delay_between_interstitial(30)

-- Show interstitial
local placement = "level_complete"  -- optional
bridge.advertisement.show_interstitial(placement)
```

### Interstitial State

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.advertisement.on("interstitial_state_changed", function (_, state)
        -- States: 'loading', 'opened', 'closed', 'failed'
        print("Interstitial state changed: ", state)

        if state == "opened" then
            -- Your logic
        elseif state == "closed" or state == "failed" then
            -- Your logic
        end
    end)
end
```

**Note:** Do not call `show_interstitial()` at game start. On platforms where this is allowed, ads will be shown automatically.

## Rewarded Ads

```lua
local bridge = require("bridge.bridge")

-- Check support
local is_supported = bridge.advertisement.is_rewarded_supported()

-- Get current state: 'loading', 'opened', 'closed', 'rewarded', 'failed'
local state = bridge.advertisement.rewarded_state()

-- Get current placement
local placement = bridge.advertisement.rewarded_placement()

-- Show rewarded ad
local placement = "double_coins"  -- optional
bridge.advertisement.show_rewarded(placement)
```

### Rewarded State

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.advertisement.on("rewarded_state_changed", function (_, state)
        -- States: 'loading', 'opened', 'closed', 'rewarded', 'failed'
        print("Rewarded state changed: ", state)

        if state == "opened" then
            -- Your logic
        elseif state == "rewarded" then
            -- Grant reward to player
        elseif state == "closed" or state == "failed" then
            -- Your logic
        end
    end)
end
```

**Important:** Only grant rewards when state is `rewarded`.

## In-App Purchases

```lua
local bridge = require("bridge.bridge")

-- Check support
local is_supported = bridge.payments.is_supported()
```

### Purchase Item

```lua
local bridge = require("bridge.bridge")

function init(self)
    local id = "coins_100"
    bridge.payments.purchase(id, function (_, purchase)
        -- success
        print("Purchased: ", purchase.id)
    end, function (_, error)
        -- error
    end)
end
```

### Consume Purchase

```lua
local bridge = require("bridge.bridge")

function init(self)
    local id = "coins_100"
    bridge.payments.consume_purchase(id, function (_, purchase)
        -- success
    end, function ()
        -- error
    end)
end
```

### Get Catalog

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.payments.get_catalog(function (_, catalog)
        for key, value in pairs(catalog) do
            local id = value.id
            local price = value.price
            local priceCurrencyCode = value.priceCurrencyCode
            local priceValue = value.priceValue
            print("Product: ", id, " Price: ", price)
        end
    end, function ()
        -- error
    end)
end
```

### Get Purchases

```lua
local bridge = require("bridge.bridge")

function init(self)
    bridge.payments.get_purchases(function (_, purchases)
        for key, value in pairs(purchases) do
            local id = value.id
            print("Owned: ", id)
        end
    end, function ()
        -- error
    end)
end
```

## Leaderboards

```lua
local bridge = require("bridge.bridge")

-- Leaderboard type: 'not_available', 'in_game', 'native', 'native_popup'
local leaderboard_type = bridge.leaderboards.type
```

### Set Score

```lua
local bridge = require("bridge.bridge")

function init(self)
    local leaderboardId = "high_score"
    local score = 1000

    bridge.leaderboards.set_score(leaderboardId, score, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

### Get Entries

Works only when `bridge.leaderboards.type` equals `in_game`:

```lua
local bridge = require("bridge.bridge")

function init(self)
    local leaderboardId = "high_score"
    local options = {}

    bridge.leaderboards.get_entries(options, function (_, data)
        for key, value in pairs(data) do
            print("ID: ", value.id)
            print("Name: ", value.name)
            print("Photo: ", value.photo)
            print("Score: ", value.score)
            print("Rank: ", value.rank)
        end
    end, function ()
        -- error
    end)
end
```

### Show Native Popup

Works only when `bridge.leaderboards.type` equals `native_popup`:

```lua
local bridge = require("bridge.bridge")

function init(self)
    local leaderboardId = "high_score"

    bridge.leaderboards.show_native_popup(leaderboardId, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

## Social Features

### Share

```lua
local bridge = require("bridge.bridge")

function init(self)
    -- Check support
    local is_supported = bridge.social.is_share_supported()

    local options = {
        vk = {
            link = "YOUR_LINK"
        },
        facebook = {
            image = "A base64 encoded image to be shared",
            text = "A text message to be shared."
        },
        msn = {
            title = "A title to display",
            image = "A base64 encoded image or image URL to be shared",
            text = "A text message to be shared."
        }
    }

    bridge.social.share(options, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

### Invite Friends

```lua
local bridge = require("bridge.bridge")

function init(self)
    local options = {
        vk = {
            text = "Hello World!"
        },
        facebook = {
            image = "A base64 encoded image to be shared",
            text = "A text message"
        }
    }

    bridge.social.invite_friends(options, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

### Join Community

```lua
local bridge = require("bridge.bridge")

function init(self)
    local options = {
        vk = {
            groupId = "YOUR_GROUP_ID"
        },
        ok = {
            groupId = "YOUR_GROUP_ID"
        },
        facebook = {
            isPage = true
        }
    }

    bridge.social.join_community(options, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

### Create Post

```lua
local bridge = require("bridge.bridge")

function init(self)
    local options = {
        ok = {
            media = {
                {
                    type = "text",
                    text = "Hello World!"
                },
                {
                    type = "link",
                    url = "https://apiok.ru"
                },
                {
                    type = "poll",
                    question = "Do you like our game?",
                    answers = {
                        { text = "Yes" },
                        { text = "No" }
                    },
                    options = "SingleChoice,AnonymousVoting"
                }
            }
        }
    }

    bridge.social.create_post(options, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

### Add to Favorites

```lua
bridge.social.add_to_favorites(function ()
    -- success
end, function ()
    -- error
end)
```

### Add to Home Screen

```lua
bridge.social.add_to_home_screen(function ()
    -- success
end, function ()
    -- error
end)
```

### Rate Game

```lua
bridge.social.rate(function ()
    -- success
end, function ()
    -- error
end)
```

## Achievements

```lua
local bridge = require("bridge.bridge")

-- Check support
local is_supported = bridge.achievements.is_supported()
local is_get_list_supported = bridge.achievements.is_get_list_supported()
local is_native_popup_supported = bridge.achievements.is_native_popup_supported()
```

### Unlock Achievement

```lua
local bridge = require("bridge.bridge")

function init(self)
    local options = {
        y8 = {
            achievement = "ACHIEVEMENT_NAME",
            achievementkey = "ACHIEVEMENT_KEY"
        },
        lagged = {
            achievement = "ACHIEVEMENT_ID"
        }
    }

    bridge.achievements.unlock(options, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

### Get List

```lua
local bridge = require("bridge.bridge")

function init(self)
    local options = {}
    bridge.achievements.get_list(options, function (_, data)
        for key, value in pairs(data) do
            print("Achievement: ", value)
        end
    end, function ()
        -- error
    end)
end
```

### Show Native Popup

```lua
local bridge = require("bridge.bridge")

function init(self)
    local options = {}
    bridge.achievements.show_native_popup(options, function ()
        -- success
    end, function ()
        -- error
    end)
end
```

## Best Practices

1. Always call `bridge.platform.send_message("game_ready")` when game is fully loaded
2. Subscribe to `audio_state_changed` and mute/unmute game audio accordingly
3. Subscribe to `pause_state_changed` and pause/resume game accordingly
4. Show ads at natural breakpoints (level transitions, player death), not during gameplay
5. Check `is_supported` methods before using platform-specific features
6. Grant rewards only when rewarded ad state is `rewarded`
7. Use platform-specific options for social features based on `bridge.platform.id()`
8. Wait for storage callbacks before making sequential storage operations
9. Do not call `show_interstitial()` at game start - it will be shown automatically where allowed
