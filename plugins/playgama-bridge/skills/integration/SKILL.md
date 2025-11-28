---
name: integration
description: Use this skill when working with Playgama Bridge SDK for HTML5 games. Activates for game development involving cross-platform publishing, advertisements, in-app purchases, leaderboards, or social features using Playgama Bridge.
---

# Playgama Bridge SDK Integration

Playgama Bridge is a cross-platform SDK for publishing HTML5 games across 20+ platforms including Playgama, YouTube, Yandex Games, Crazy Games, Poki, Facebook, Telegram, and more.

## Installation

Add the SDK script to the HTML `<head>`:

```html
<script src="https://bridge.playgama.com/v1/stable/playgama-bridge.js"></script>
```

CDN options:
- `https://bridge.playgama.com/v1/stable/playgama-bridge.js` - Recommended (latest v1.x.x)
- `https://bridge.playgama.com/latest/playgama-bridge.js` - Bleeding edge
- `https://bridge.playgama.com/v1.26.0/playgama-bridge.js` - Specific version

## Initialization

```javascript
bridge.initialize()
    .then(() => {
        // SDK ready
    })
    .catch(error => {
        // Handle error
    })
```

After game is fully loaded and ready for player interaction:

```javascript
bridge.platform.sendMessage('game_ready')
```

Other messages:
```javascript
bridge.platform.sendMessage('in_game_loading_started')
bridge.platform.sendMessage('in_game_loading_stopped')
bridge.platform.sendMessage('gameplay_started')
bridge.platform.sendMessage('gameplay_stopped')
bridge.platform.sendMessage('player_got_achievement')
```

## Device

```javascript
// Device type
bridge.device.type // 'mobile', 'tablet', 'desktop', 'tv'
```

## Platform Detection

```javascript
// Platform ID
bridge.platform.id
// Values: 'playgama', 'facebook', 'crazy_games', 'mock', etc.

// User language (ISO 639-1)
bridge.platform.language // 'en', 'ru', etc.

// Top-level domain
bridge.platform.tld // 'com', 'ru', null

// URL payload
bridge.platform.payload

// Server time (UTC milliseconds)
bridge.platform.getServerTime().then(result => console.log(result))
```

## Player

```javascript
// Player ID (null if not authorized)
bridge.player.id

// Player name (null if unavailable)
bridge.player.name

// Player photos (array sorted by increasing resolution)
bridge.player.photos

// Platform-specific player data for verification
bridge.player.extra
```

## Authorization

```javascript
// Check if authorization is supported
bridge.player.isAuthorizationSupported

// Check if player is authorized
bridge.player.isAuthorized

// Authorize player
let options = {}
bridge.player.authorize(options)
    .then(() => {
        // Player successfully authorized
    })
    .catch(error => {
        // Authorization failed or cancelled
    })
```

## Audio & Pause State

```javascript
// Audio state
bridge.platform.isAudioEnabled

bridge.platform.on(bridge.EVENT_NAME.AUDIO_STATE_CHANGED, isEnabled => {
    // Mute/unmute game audio
})

// Pause state
bridge.platform.on(bridge.EVENT_NAME.PAUSE_STATE_CHANGED, isPaused => {
    // Pause/unpause game
})
```

## Storage

```javascript
// Get single value
bridge.storage.get('key')
    .then(data => console.log(data))

// Get multiple values
bridge.storage.get(['key1', 'key2'])
    .then(data => console.log(data))

// Set single value
bridge.storage.set('key', 'value')
    .then(() => { /* saved */ })

// Set multiple values
bridge.storage.set(['key1', 'key2'], ['value1', 'value2'])
    .then(() => { /* saved */ })

// Delete
bridge.storage.delete('key')
bridge.storage.delete(['key1', 'key2'])
```

## Banner Ads

```javascript
// Check support
bridge.advertisement.isBannerSupported

// Show banner
let position = 'bottom' // 'top' or 'bottom'
let placement = 'menu' // optional
bridge.advertisement.showBanner(position, placement)

// Hide banner
bridge.advertisement.hideBanner()

// Banner state
bridge.advertisement.bannerState // 'loading', 'shown', 'hidden', 'failed'

bridge.advertisement.on(bridge.EVENT_NAME.BANNER_STATE_CHANGED, state => {
    console.log('Banner state:', state)
})
```

## Interstitial Ads

```javascript
// Check support
bridge.advertisement.isInterstitialSupported

// Show interstitial
let placement = 'level_complete' // optional
bridge.advertisement.showInterstitial(placement)

// Minimum delay between interstitials (default: 60 seconds)
bridge.advertisement.minimumDelayBetweenInterstitial
bridge.advertisement.setMinimumDelayBetweenInterstitial(30)

// State tracking
bridge.advertisement.interstitialState // 'loading', 'opened', 'closed', 'failed'

bridge.advertisement.on(bridge.EVENT_NAME.INTERSTITIAL_STATE_CHANGED, state => {
    if (state === 'opened') {
        // Your logic
    } else if (state === 'closed' || state === 'failed') {
        // Your logic
    }
})
```

## Rewarded Ads

```javascript
// Check support
bridge.advertisement.isRewardedSupported

// Show rewarded ad
let placement = 'double_coins' // optional
bridge.advertisement.showRewarded(placement)

// State tracking
bridge.advertisement.rewardedState
// States: 'loading', 'opened', 'closed', 'rewarded', 'failed'

bridge.advertisement.on(bridge.EVENT_NAME.REWARDED_STATE_CHANGED, state => {
    if (state === 'opened') {
        // Your logic
    } else if (state === 'rewarded') {
        // Grant reward to player
    } else if (state === 'closed' || state === 'failed') {
        // Your logic
    }
})

// Current placement
bridge.advertisement.rewardedPlacement
```

## In-App Purchases

```javascript
// Check support
bridge.payments.isSupported

// Get catalog
bridge.payments.getCatalog()
    .then(items => {
        items.forEach(item => {
            console.log('ID:', item.id)
            console.log('Price:', item.price)
            console.log('Currency:', item.priceCurrencyCode)
            console.log('Value:', item.priceValue)
        })
    })

// Purchase
bridge.payments.purchase('product_id')
    .then(purchase => console.log('Purchased:', purchase.id))
    .catch(error => { /* cancelled or failed */ })

// Consume purchase (for consumable items)
bridge.payments.consumePurchase('product_id')
    .then(purchase => console.log('Consumed:', purchase.id))

// Get purchased items
bridge.payments.getPurchases()
    .then(purchases => {
        purchases.forEach(p => console.log('Owned:', p.id))
    })
```

Config example (playgama-bridge-config.json):
```json
{
    "payments": [
        {
            "id": "coins_100",
            "playgama": { "amount": 1 },
            "playdeck": { "amount": 1, "description": "100 Coins" }
        }
    ]
}
```

## Leaderboards

```javascript
// Check type
bridge.leaderboards.type
// 'not_available', 'in_game', 'native', 'native_popup'

// Set score
bridge.leaderboards.setScore('leaderboard_id', 1000)
    .then(() => { /* saved */ })

// Get entries (only when type = 'in_game')
bridge.leaderboards.getEntries('leaderboard_id')
    .then(entries => {
        entries.forEach(entry => {
            console.log('Name:', entry.name)
            console.log('Score:', entry.score)
            console.log('Rank:', entry.rank)
            console.log('Photo:', entry.photo)
        })
    })

// Show native popup (only when type = 'native_popup')
bridge.leaderboards.showNativePopup('leaderboard_id')
```

Config example:
```json
{
    "leaderboards": [
        {
            "id": "high_score"
        }
    ]
}
```

## Social Features

### Share
```javascript
bridge.social.isShareSupported

let options = {}
switch (bridge.platform.id) {
    case 'vk':
        options = { link: 'https://...' }
        break
    case 'facebook':
        options = { image: 'base64...', text: 'Check this game!' }
        break
}
bridge.social.share(options)
```

### Invite Friends
```javascript
bridge.social.isInviteFriendsSupported

bridge.social.inviteFriends({ text: 'Join me!' })
```

### Join Community
```javascript
bridge.social.isJoinCommunitySupported

let options = {}
switch (bridge.platform.id) {
    case 'vk':
    case 'ok':
        options = { groupId: 123456 }
        break
}
bridge.social.joinCommunity(options)
```

### Other Social Methods
```javascript
// Add to favorites
bridge.social.isAddToFavoritesSupported
bridge.social.addToFavorites()

// Rate game
bridge.social.isRateSupported
bridge.social.rate()

// Add to home screen
bridge.social.isAddToHomeScreenSupported
bridge.social.addToHomeScreen()

// Create post
bridge.social.isCreatePostSupported
bridge.social.createPost(options)

// Check external links
bridge.social.isExternalLinksAllowed
```

## Best Practices

1. Always call `bridge.platform.sendMessage('game_ready')` when game is loaded
2. Mute/restore game audio when audio state changed
3. Show ads at natural breakpoints (level transitions, player death), not during gameplay
4. Check `isSupported` before using features
5. Handle pause state changes to pause/resume game
6. Use platform-specific options for social features
7. Grant rewards only when rewarded ad state is 'rewarded'
