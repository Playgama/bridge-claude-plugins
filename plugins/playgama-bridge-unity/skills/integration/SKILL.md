---
name: integration
description: Use this skill when working with Playgama Bridge SDK for Unity games. Activates for game development involving cross-platform publishing, advertisements, in-app purchases, leaderboards, or social features using Playgama Bridge in Unity/C#.
---

# Playgama Bridge SDK Integration for Unity

Playgama Bridge is a cross-platform SDK for publishing Unity WebGL games across 20+ platforms including Playgama, YouTube, Yandex Games, Crazy Games, Poki, Facebook, Telegram, and more.

## Installation

1. Open Unity Package Manager (Window -> Package Manager)
2. Click "+" and select "Add package from git URL"
3. Enter: `https://github.com/playgama/bridge-unity.git`
4. Click "Add"

## WebGL Export

When building for WebGL, select the appropriate index template provided by the SDK to ensure proper initialization.

## Initialization

The SDK initializes automatically. Access all modules through the static `Bridge` class:

```csharp
using Playgama;

// All modules are accessible via Bridge static properties
Bridge.advertisement
Bridge.game
Bridge.storage
Bridge.platform
Bridge.social
Bridge.player
Bridge.device
Bridge.leaderboards
Bridge.payments
Bridge.achievements
Bridge.remoteConfig
```

## Device

```csharp
// Device type
DeviceType type = Bridge.device.type;
// Values: DeviceType.Mobile, DeviceType.Tablet, DeviceType.Desktop, DeviceType.TV
```

## Platform

```csharp
// Platform ID
string platformId = Bridge.platform.id;
// Values: "playgama", "facebook", "crazy_games", "yandex", "vk", etc.

// User language (ISO 639-1)
string language = Bridge.platform.language; // "en", "ru", etc.

// Top-level domain
string tld = Bridge.platform.tld; // "com", "ru", null

// URL payload
string payload = Bridge.platform.payload;

// Audio state
bool isAudioEnabled = Bridge.platform.isAudioEnabled;

// Server time
Bridge.platform.GetServerTime((DateTime? serverTime) =>
{
    if (serverTime.HasValue) {
        Debug.Log($"Server time: {serverTime.Value}");
    }
});

// Send message to platform
Bridge.platform.SendMessage(PlatformMessage.GameReady);
Bridge.platform.SendMessage(PlatformMessage.InGameLoadingStarted);
Bridge.platform.SendMessage(PlatformMessage.InGameLoadingStopped);
Bridge.platform.SendMessage(PlatformMessage.GameplayStarted);
Bridge.platform.SendMessage(PlatformMessage.GameplayStopped);
Bridge.platform.SendMessage(PlatformMessage.PlayerGotAchievement);

// Audio state changed event
Bridge.platform.audioStateChanged += (bool isEnabled) =>
{
    // Mute/unmute game audio
    AudioListener.volume = isEnabled ? 1f : 0f;
};

// Pause state changed event
Bridge.platform.pauseStateChanged += (bool isPaused) =>
{
    // Pause/resume game
    Time.timeScale = isPaused ? 0f : 1f;
};
```

## Player

```csharp
// Player ID (null if not authorized)
string playerId = Bridge.player.id;

// Player name (null if unavailable)
string playerName = Bridge.player.name;

// Player photos (list of URLs sorted by increasing resolution)
List<string> photos = Bridge.player.photos;

// Platform-specific player data
Dictionary<string, string> extra = Bridge.player.extra;

// Check if authorization is supported
bool authSupported = Bridge.player.isAuthorizationSupported;

// Check if player is authorized
bool isAuthorized = Bridge.player.isAuthorized;

// Authorize player
var options = new Dictionary<string, object>();
Bridge.player.Authorize(options, (bool success) =>
{
    if (success)
    {
        Debug.Log("Player authorized");
        Debug.Log($"Player ID: {Bridge.player.id}");
        Debug.Log($"Player name: {Bridge.player.name}");
    }
    else
    {
        Debug.Log("Authorization failed or cancelled");
    }
});
```

## Storage

```csharp
// Get single value
Bridge.storage.Get("key", (bool success, string value) =>
{
    if (success) {
        Debug.Log($"Value: {value}");
    }
});

// Get multiple values
var keys = new List<string> { "key1", "key2" };
Bridge.storage.Get(keys, (bool success, List<string> values) =>
{
    if (success)
    {
        for (int i = 0; i < keys.Count; i++) {
            Debug.Log($"{keys[i]}: {values[i]}");
        }
    }
});

// Set single value (string)
Bridge.storage.Set("key", "value", (bool success) =>
{
    if (success) {
        Debug.Log("Saved");
    }
});

// Set single value (int)
Bridge.storage.Set("score", 100, (bool success) =>
{
    if (success) {
        Debug.Log("Score saved");
    }
});

// Set single value (bool)
Bridge.storage.Set("tutorial_complete", true);

// Set multiple values
var saveKeys = new List<string> { "key1", "key2" };
var saveValues = new List<object> { "value1", 42 };
Bridge.storage.Set(saveKeys, saveValues, (bool success) =>
{
    if (success) {
        Debug.Log("All values saved");
    }
});

// Delete single key
Bridge.storage.Delete("key", (bool success) =>
{
    if (success) {
        Debug.Log("Deleted");
    }
});

// Delete multiple keys
Bridge.storage.Delete(new List<string> { "key1", "key2" });
```

## Banner Ads

```csharp
// Check support
bool supported = Bridge.advertisement.isBannerSupported;

// Show banner
Bridge.advertisement.ShowBanner(BannerPosition.Bottom, "menu");

// Hide banner
Bridge.advertisement.HideBanner();

// Banner state
BannerState state = Bridge.advertisement.bannerState;
// Values: BannerState.Loading, BannerState.Shown, BannerState.Hidden, BannerState.Failed

// State changed event
Bridge.advertisement.bannerStateChanged += (BannerState state) =>
{
    Debug.Log($"Banner state: {state}");
};
```

## Interstitial Ads

```csharp
// Check support
bool supported = Bridge.advertisement.isInterstitialSupported;

// Get/set minimum delay between interstitials (default: 60 seconds)
int delay = Bridge.advertisement.minimumDelayBetweenInterstitial;
Bridge.advertisement.SetMinimumDelayBetweenInterstitial(30);

// Show interstitial
Bridge.advertisement.ShowInterstitial("level_complete");

// Interstitial state
InterstitialState state = Bridge.advertisement.interstitialState;
// Values: InterstitialState.Loading, InterstitialState.Opened, InterstitialState.Closed, InterstitialState.Failed

// State changed event
Bridge.advertisement.interstitialStateChanged += (InterstitialState state) =>
{
    switch (state)
    {
        case InterstitialState.Opened:
            // Your logic
            break;
        case InterstitialState.Closed:
        case InterstitialState.Failed:
            // Your logic
            break;
    }
};
```

## Rewarded Ads

```csharp
// Check support
bool supported = Bridge.advertisement.isRewardedSupported;

// Show rewarded ad
Bridge.advertisement.ShowRewarded("double_coins");

// Current placement
string placement = Bridge.advertisement.rewardedPlacement;

// Rewarded state
RewardedState state = Bridge.advertisement.rewardedState;
// Values: RewardedState.Loading, RewardedState.Opened, RewardedState.Closed, RewardedState.Rewarded, RewardedState.Failed

// State changed event
Bridge.advertisement.rewardedStateChanged += (RewardedState state) =>
{
    switch (state)
    {
        case RewardedState.Opened:
            // Your logic
            break;
        case RewardedState.Rewarded:
            // Grant reward to player
            break;
        case RewardedState.Closed:
        case RewardedState.Failed:
            // Your logic
            break;
    }
};
```

## AdBlock Detection

```csharp
Bridge.advertisement.CheckAdBlock((bool isAdBlockEnabled) =>
{
    if (isAdBlockEnabled) {
        Debug.Log("AdBlock detected");
    }
});
```

## In-App Purchases

```csharp
// Check support
bool supported = Bridge.payments.isSupported;

// Get catalog
Bridge.payments.GetCatalog((bool success, List<Dictionary<string, string>> items) =>
{
    if (success)
    {
        foreach (var item in items)
        {
            Debug.Log($"ID: {item["id"]}");
            Debug.Log($"Price: {item["price"]}");
            Debug.Log($"Currency: {item["priceCurrencyCode"]}");
            Debug.Log($"Value: {item["priceValue"]}");
        }
    }
});

// Purchase
Bridge.payments.Purchase("product_id", (bool success, Dictionary<string, string> purchase) =>
{
    if (success)
    {
        Debug.Log($"Purchased: {purchase["id"]}");
        // Grant item to player
    }
    else
    {
        Debug.Log("Purchase cancelled or failed");
    }
});

// Purchase with options (options is optional)
var options = new Dictionary<string, object>
{
    { "externalId", "test_externalId" }
};
Bridge.payments.Purchase("product_id", options, (bool success, Dictionary<string, string> purchase) =>
{
    if (success) {
        Debug.Log($"Purchased: {purchase["id"]}");
    }
});

// Consume purchase (for consumable items)
Bridge.payments.ConsumePurchase("product_id", (bool success, Dictionary<string, string> purchase) =>
{
    if (success) {
        Debug.Log($"Consumed: {purchase["id"]}");
    }
});

// Get purchased items
Bridge.payments.GetPurchases((bool success, List<Dictionary<string, string>> purchases) =>
{
    if (success)
    {
        foreach (var p in purchases) {
            Debug.Log($"Owned: {p["id"]}");
        }
    }
});
```

## Leaderboards

```csharp
// Check type
LeaderboardType type = Bridge.leaderboards.type;
// Values: LeaderboardType.NotAvailable, LeaderboardType.InGame, LeaderboardType.Native, LeaderboardType.NativePopup

// Set score
Bridge.leaderboards.SetScore("leaderboard_id", 1000, (bool success) =>
{
    if (success) {
        Debug.Log("Score saved");
    }
});

// Get entries (only when type = InGame)
Bridge.leaderboards.GetEntries("leaderboard_id", (bool success, List<Dictionary<string, string>> entries) =>
{
    if (success)
    {
        foreach (var entry in entries)
        {
            Debug.Log($"Name: {entry["name"]}");
            Debug.Log($"Score: {entry["score"]}");
            Debug.Log($"Rank: {entry["rank"]}");
            Debug.Log($"Photo: {entry["photo"]}");
        }
    }
});

// Show native popup (only when type = NativePopup)
Bridge.leaderboards.ShowNativePopup("leaderboard_id", (bool success) =>
{
    Debug.Log($"Popup closed, success: {success}");
});
```

## Social Features

### Share
```csharp
bool supported = Bridge.social.isShareSupported;

var options = new Dictionary<string, object>();
switch (Bridge.platform.id)
{
    case "vk":
        options["link"] = "https://...";
        break;
    case "facebook":
        options["image"] = "base64...";
        options["text"] = "Check this game!";
        break;
}

Bridge.social.Share(options, (bool success) =>
{
    Debug.Log($"Share success: {success}");
});
```

### Invite Friends
```csharp
bool supported = Bridge.social.isInviteFriendsSupported;

var options = new Dictionary<string, object>
{
    { "text", "Join me in this game!" }
};
Bridge.social.InviteFriends(options, (bool success) =>
{
    Debug.Log($"Invite success: {success}");
});
```

### Join Community
```csharp
bool supported = Bridge.social.isJoinCommunitySupported;

var options = new Dictionary<string, object>();
switch (Bridge.platform.id)
{
    case "vk":
    case "ok":
        options["groupId"] = 123456;
        break;
}
Bridge.social.JoinCommunity(options, (bool success) =>
{
    Debug.Log($"Join community success: {success}");
});
```

### Other Social Methods
```csharp
// Add to favorites
if (Bridge.social.isAddToFavoritesSupported)
{
    Bridge.social.AddToFavorites((bool success) =>
    {
        Debug.Log($"Add to favorites: {success}");
    });
}

// Rate game
if (Bridge.social.isRateSupported)
{
    Bridge.social.Rate((bool success) =>
    {
        Debug.Log($"Rate: {success}");
    });
}

// Add to home screen
if (Bridge.social.isAddToHomeScreenSupported)
{
    Bridge.social.AddToHomeScreen((bool success) =>
    {
        Debug.Log($"Add to home screen: {success}");
    });
}

// Create post
if (Bridge.social.isCreatePostSupported)
{
    var options = new Dictionary<string, object>
    {
        { "text", "I scored 1000 points!" }
    };
    Bridge.social.CreatePost(options, (bool success) =>
    {
        Debug.Log($"Create post: {success}");
    });
}

// Check external links
bool externalLinksAllowed = Bridge.social.isExternalLinksAllowed;
```

## Achievements

```csharp
// Check support
bool supported = Bridge.achievements.isSupported;
bool getListSupported = Bridge.achievements.isGetListSupported;
bool nativePopupSupported = Bridge.achievements.isNativePopupSupported;

// Unlock achievement
var options = new Dictionary<string, object>
{
    { "id", "first_win" }
};
Bridge.achievements.Unlock(options, (bool success) =>
{
    Debug.Log($"Achievement unlocked: {success}");
});

// Get achievements list
Bridge.achievements.GetList(new Dictionary<string, object>(), (bool success, List<Dictionary<string, string>> achievements) =>
{
    if (success)
    {
        foreach (var achievement in achievements)
        {
            Debug.Log($"ID: {achievement["id"]}");
            Debug.Log($"Name: {achievement["name"]}");
            Debug.Log($"Unlocked: {achievement["unlocked"]}");
        }
    }
});

// Show native popup
Bridge.achievements.ShowNativePopup(new Dictionary<string, object>(), (bool success) =>
{
    Debug.Log($"Popup closed: {success}");
});
```

## Remote Config

```csharp
// Check support
bool supported = Bridge.remoteConfig.isSupported;

// Get remote config
Bridge.remoteConfig.Get((bool success, Dictionary<string, string> config) =>
{
    if (success)
    {
        foreach (var kvp in config)
        {
            Debug.Log($"{kvp.Key}: {kvp.Value}");
        }
    }
});

// Get with options
var options = new Dictionary<string, object>
{
    { "key", "value" }
};
Bridge.remoteConfig.Get(options, (bool success, Dictionary<string, string> config) =>
{
    if (success)
    {
        string someValue = config["some_key"];
    }
});
```

## Best Practices

1. Always call `Bridge.platform.SendMessage(PlatformMessage.GameReady)` when game is fully loaded and ready
2. Subscribe to `audioStateChanged` and mute/unmute game audio accordingly
3. Subscribe to `pauseStateChanged` and pause/resume game accordingly
4. Show ads at natural breakpoints (level transitions, player death), not during gameplay
5. Check `isSupported` properties before using platform-specific features
6. Grant rewards only when `RewardedState.Rewarded` is received
7. Use platform-specific options for social features