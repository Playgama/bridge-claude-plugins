---
name: integration
description: Use this skill when working with Playgama Bridge SDK for Godot games. Activates for game development involving cross-platform publishing, advertisements, in-app purchases, leaderboards, or social features using Playgama Bridge in Godot/GDScript.
---

# Playgama Bridge SDK Integration for Godot

Playgama Bridge is a cross-platform SDK for publishing Godot HTML5 games across 20+ platforms including Playgama, YouTube, Yandex Games, Crazy Games, Poki, Facebook, Telegram, Xiaomi, and more.

## Installation

### Godot 3
1. Download `playgama_bridge.zip` from [GitHub releases](https://github.com/playgama/bridge-godot/releases)
2. Extract and place contents into `res://addons/` directory
3. Open Project Settings -> Plugins tab
4. Enable "playgama_bridge" plugin
5. Ensure Bridge plugin appears first in AutoLoad list

### Godot 4
1. Download from [GitHub releases for Godot 4](https://github.com/playgama/bridge-godot-4/releases)
2. Follow same steps as Godot 3

## Configuration

Edit `addons/playgama_bridge/template/playgama-bridge-config.json` for game settings (identifiers, in-app purchases, leaderboards).

## HTML5 Export

When exporting to HTML5, set Custom HTML Shell to: `res://addons/playgama_bridge/template/index.html`

## Initialization

The SDK initializes automatically. Access all modules through the `Bridge` singleton:

```gdscript
Bridge.platform
Bridge.player
Bridge.device
Bridge.storage
Bridge.advertisement
Bridge.payments
Bridge.leaderboards
Bridge.social
Bridge.achievements
```

After game is fully loaded:

```gdscript
Bridge.platform.send_message("game_ready")
```

## Device

```gdscript
# Device type: 'mobile', 'tablet', 'desktop', 'tv'
Bridge.device.type
```

## Platform

```gdscript
# Platform ID: 'playgama', 'vk', 'yandex', 'crazy_games', etc.
Bridge.platform.id

# User language (ISO 639-1)
Bridge.platform.language  # 'en', 'ru', etc.

# Top-level domain
Bridge.platform.tld  # 'com', 'ru', null

# URL payload
Bridge.platform.payload

# Audio state
Bridge.platform.is_audio_enabled

# Server time (Godot 4)
func _ready():
    Bridge.platform.get_server_time(Callable(self, "_on_get_server_time_completed"))

# Server time (Godot 3)
func _ready():
    Bridge.platform.get_server_time(funcref(self, "_on_get_server_time_completed"))

func _on_get_server_time_completed(success, time):
    if success:
        print("Server time: ", time)
```

### Platform Messages

```gdscript
Bridge.platform.send_message("game_ready")
Bridge.platform.send_message("gameplay_started")
Bridge.platform.send_message("gameplay_stopped")
Bridge.platform.send_message("in_game_loading_started")
Bridge.platform.send_message("in_game_loading_stopped")
Bridge.platform.send_message("player_got_achievement")
```

### Platform Events (Godot 4)

```gdscript
func _ready():
    Bridge.platform.connect("audio_state_changed", Callable(self, "_on_audio_changed"))
    Bridge.platform.connect("pause_state_changed", Callable(self, "_on_pause_changed"))

func _on_audio_changed(is_enabled):
    # Mute/unmute game audio
    AudioServer.set_bus_mute(0, not is_enabled)

func _on_pause_changed(is_paused):
    get_tree().paused = is_paused
```

### Platform Events (Godot 3)

```gdscript
func _ready():
    Bridge.platform.connect("audio_state_changed", self, "_on_audio_changed")
    Bridge.platform.connect("pause_state_changed", self, "_on_pause_changed")
```

## Player

```gdscript
# Player ID (null if not authorized)
Bridge.player.id

# Player name (null if unavailable)
Bridge.player.name

# Player photos (array of URLs sorted by resolution)
Bridge.player.photos

# Platform-specific player data
Bridge.player.extra

# Check authorization support
Bridge.player.is_authorization_supported

# Check if authorized
Bridge.player.is_authorized
```

### Authorization (Godot 4)

```gdscript
func _ready():
    var options = {}
    Bridge.player.authorize(options, Callable(self, "_on_authorize"))

func _on_authorize(success):
    if success:
        print("Authorized: ", Bridge.player.id)
    else:
        print("Authorization failed")
```

### Authorization (Godot 3)

```gdscript
func _ready():
    var options = {}
    Bridge.player.authorize(options, funcref(self, "_on_authorize"))

func _on_authorize(success):
    if success:
        print("Authorized")
```

## Storage

### Get Data (Godot 4)

```gdscript
func _ready():
    # Single value
    Bridge.storage.get("level", Callable(self, "_on_get"))

    # Multiple values
    Bridge.storage.get(["level", "coins"], Callable(self, "_on_get"))

func _on_get(success, data):
    if success and data != null:
        print("Data: ", data)
```

### Set Data (Godot 4)

```gdscript
func _ready():
    # Single value
    Bridge.storage.set("level", "dungeon_123", Callable(self, "_on_set"))

    # Multiple values
    Bridge.storage.set(["level", "coins"], ["dungeon_123", 42], Callable(self, "_on_set"))

func _on_set(success):
    print("Save success: ", success)
```

### Delete Data (Godot 4)

```gdscript
func _ready():
    Bridge.storage.delete("level", Callable(self, "_on_delete"))
    Bridge.storage.delete(["level", "coins"], Callable(self, "_on_delete"))

func _on_delete(success):
    print("Delete success: ", success)
```

### Storage (Godot 3)

```gdscript
func _ready():
    Bridge.storage.get("level", funcref(self, "_on_get"))
    Bridge.storage.set("level", "value", funcref(self, "_on_set"))
    Bridge.storage.delete("level", funcref(self, "_on_delete"))
```

## Banner Ads

```gdscript
# Check support
Bridge.advertisement.is_banner_supported

# Show banner
var position = Bridge.BannerPosition.BOTTOM  # TOP or BOTTOM
var placement = "menu"  # optional
Bridge.advertisement.show_banner(position, placement)

# Hide banner
Bridge.advertisement.hide_banner()
```

### Banner State (Godot 4)

```gdscript
func _ready():
    Bridge.advertisement.connect("banner_state_changed", Callable(self, "_on_banner_state_changed"))

func _on_banner_state_changed(state):
    # States: 'loading', 'shown', 'hidden', 'failed'
    print("Banner: ", state)
```

### Banner State (Godot 3)

```gdscript
func _ready():
    Bridge.advertisement.connect("banner_state_changed", self, "_on_banner_state_changed")
```

## Interstitial Ads

```gdscript
# Check support
Bridge.advertisement.is_interstitial_supported

# Set minimum delay between ads (default: 60 seconds)
Bridge.advertisement.set_minimum_delay_between_interstitial(30)

# Show interstitial
var placement = "level_complete"  # optional
Bridge.advertisement.show_interstitial(placement)
```

### Interstitial State (Godot 4)

```gdscript
func _ready():
    Bridge.advertisement.connect("interstitial_state_changed", Callable(self, "_on_interstitial_state_changed"))

func _on_interstitial_state_changed(state):
    # States: 'loading', 'opened', 'closed', 'failed'
    match state:
        "opened":
            # Your logic
            pass
        "closed", "failed":
            # Your logic
            pass
```

### Interstitial State (Godot 3)

```gdscript
func _ready():
    Bridge.advertisement.connect("interstitial_state_changed", self, "_on_interstitial_state_changed")
```

## Rewarded Ads

```gdscript
# Check support
Bridge.advertisement.is_rewarded_supported

# Current state and placement
Bridge.advertisement.rewarded_state
Bridge.advertisement.rewarded_placement

# Show rewarded ad
var placement = "double_coins"  # optional
Bridge.advertisement.show_rewarded(placement)
```

### Rewarded State (Godot 4)

```gdscript
func _ready():
    Bridge.advertisement.connect("rewarded_state_changed", Callable(self, "_on_rewarded_state_changed"))

func _on_rewarded_state_changed(state):
    # States: 'loading', 'opened', 'closed', 'rewarded', 'failed'
    match state:
        "opened":
            # Your logic
            pass
        "rewarded":
            # Grant reward to player
            pass
        "closed", "failed":
            # Your logic
            pass
```

### Rewarded State (Godot 3)

```gdscript
func _ready():
    Bridge.advertisement.connect("rewarded_state_changed", self, "_on_rewarded_state_changed")
```

## In-App Purchases

```gdscript
# Check support
Bridge.payments.is_supported
```

### Purchase (Godot 4)

```gdscript
func _ready():
    var id = "coins_100"
    Bridge.payments.purchase(id, Callable(self, "_on_purchase"))

func _on_purchase(success, purchase):
    if success:
        print("Purchased: ", purchase)
```

### Consume Purchase (Godot 4)

```gdscript
func _ready():
    Bridge.payments.consume_purchase("coins_100", Callable(self, "_on_consume"))

func _on_consume(success, purchase):
    if success:
        print("Consumed: ", purchase)
```

### Get Catalog (Godot 4)

```gdscript
func _ready():
    Bridge.payments.get_catalog(Callable(self, "_on_catalog"))

func _on_catalog(success, catalog):
    if success:
        for item in catalog:
            print("ID: ", item.id)
            print("Price: ", item.price)
            print("Price Currency Code: ", item.priceCurrencyCode)
            print("Price Value: ", item.priceValue)
```

### Get Purchases (Godot 4)

```gdscript
func _ready():
    Bridge.payments.get_purchases(Callable(self, "_on_purchases"))

func _on_purchases(success, purchases):
    if success:
        for p in purchases:
            print("Owned: ", p.id)
```

### Payments (Godot 3)

```gdscript
func _ready():
    Bridge.payments.purchase("coins_100", funcref(self, "_on_purchase"))
    Bridge.payments.get_catalog(funcref(self, "_on_catalog"))
```

## Leaderboards

```gdscript
# Leaderboard type: 'not_available', 'in_game', 'native', 'native_popup'
Bridge.leaderboards.type
```

### Set Score (Godot 4)

```gdscript
func _ready():
    var leaderboard_id = "high_score"
    var score = 1000
    Bridge.leaderboards.set_score(leaderboard_id, score, Callable(self, "_on_set_score_completed"))

func _on_set_score_completed(success):
    print("Score saved: ", success)
```

### Get Entries (Godot 4)

Works only when `Bridge.leaderboards.type` equals `in_game`:

```gdscript
func _ready():
    Bridge.leaderboards.get_entries("high_score", Callable(self, "_on_get_entries_completed"))

func _on_get_entries_completed(success, entries):
    if success:
        for entry in entries:
            print("Name: ", entry.name)
            print("Score: ", entry.score)
            print("Rank: ", entry.rank)
            print("Photo: ", entry.photo)
```

### Show Native Popup (Godot 4)

Works only when type equals `native_popup`:

```gdscript
func _ready():
    Bridge.leaderboards.show_native_popup("high_score", Callable(self, "_on_popup"))

func _on_popup(success):
    print("Popup closed")
```

### Leaderboards (Godot 3)

```gdscript
func _ready():
    Bridge.leaderboards.set_score("high_score", 1000, funcref(self, "_on_set_score"))
```

## Social Features

### Share (Godot 4)

```gdscript
func _ready():
    var options = {}
    match Bridge.platform.id:
        "vk":
            options = {"link": "https://..."}
        "facebook":
            options = {"image": "base64...", "text": "Check this game!"}

    Bridge.social.share(options, Callable(self, "_on_share"))

func _on_share(success):
    print("Share: ", success)
```

### Invite Friends (Godot 4)

```gdscript
func _ready():
    var options = {}
    match Bridge.platform.id:
        "ok":
            options = {"text": "Join me!"}
        "facebook":
            options = {"image": "base64...", "text": "Play with me!"}

    Bridge.social.invite_friends(options, Callable(self, "_on_invite"))

func _on_invite(success):
    print("Invite: ", success)
```

### Join Community (Godot 4)

```gdscript
func _ready():
    var options = {}
    match Bridge.platform.id:
        "vk", "ok":
            options = {"groupId": 123456}
        "facebook":
            options = {"isPage": true}

    Bridge.social.join_community(options, Callable(self, "_on_join"))

func _on_join(success):
    print("Join community: ", success)
```

### Add to Favorites (Godot 4)

```gdscript
func _ready():
    Bridge.social.add_to_favorites(Callable(self, "_on_favorites"))

func _on_favorites(success):
    print("Added to favorites: ", success)
```

### Social (Godot 3)

```gdscript
func _ready():
    Bridge.social.share(options, funcref(self, "_on_share"))
    Bridge.social.invite_friends(options, funcref(self, "_on_invite"))
```

## Achievements

```gdscript
# Check support
Bridge.achievements.is_supported
Bridge.achievements.is_get_list_supported
Bridge.achievements.is_native_popup_supported
```

### Unlock Achievement (Godot 4)

```gdscript
func _ready():
    var options = {}
    match Bridge.platform.id:
        "y8":
            options = {
                "achievementkey": "YOUR_KEY",
                "achievement": "YOUR_NAME"
            }
        "lagged":
            options = {"achievement": "YOUR_ID"}

    Bridge.achievements.unlock(options, Callable(self, "_on_unlock"))

func _on_unlock(success):
    print("Unlocked: ", success)
```

### Get List (Godot 4)

```gdscript
func _ready():
    Bridge.achievements.get_list(Callable(self, "_on_list"))

func _on_list(success, achievements):
    if success:
        for a in achievements:
            print("Achievement: ", a)
```

### Show Native Popup (Godot 4)

```gdscript
func _ready():
    Bridge.achievements.show_native_popup(Callable(self, "_on_popup"))

func _on_popup(success):
    print("Popup closed")
```

### Achievements (Godot 3)

```gdscript
func _ready():
    Bridge.achievements.unlock(options, funcref(self, "_on_unlock"))
```

## Best Practices

1. Always call `Bridge.platform.send_message("game_ready")` when game is fully loaded
2. Connect to `audio_state_changed` and mute/unmute game audio accordingly
3. Connect to `pause_state_changed` and pause/resume game accordingly
4. Show ads at natural breakpoints (level transitions, player death), not during gameplay
5. Check `is_supported` properties before using platform-specific features
6. Grant rewards only when rewarded ad state is `rewarded`
7. Use platform-specific options for social features
8. Wait for storage callbacks before making sequential storage operations
