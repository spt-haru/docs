# EFT Server documentation

By Senko-san, SlejmUr, King and Chomp.

This is a rough documentation of EFT's server endpoints, data and how it's
processed. This includes historical data and background information relevant
to make EFT's "offline raid" actually function offline.

## Logging in

Before version 0.11.7.4xxx, BSGLauncher generated a login token:

```jsonc
{
    "email": "john@example.com",
    "password": "mypassword",
    "toggle": true,
    "timestamp": 0  // unix timestamp
}
```

Then the login token got converted to `base64`, and finally to bytes.
The token got stored in `HKCU\\SOFTWARE\\Battlestate Games\\EscapeFromTarkov`
under the entry `bC5vLmcuaS5u_h1472614626` as `REG_BINARY`.

In order to connect to a different server than EFT's own server, you could
modify `<gamedir>/client.config.json` and set `backendUrl` to your desired
endpoint.

Starting version 0.11.4xxx, EFT allowed both token storage in registry as
launch arguments for starting the game:

```
EscapeFromTarkov.exe -token=<LoginTokenBase64> -config={"backendUrl":"http://prod.escapefromtarkov.com","version":"live"}
```

The config in the launch arugments is in the following format:

```jsonc
{
    "backendUrl": "", // http(s) endpoint
    "version": ""     // access (either "live" or "dev")
}
```

Starting 0.12.0, using `client.config.json` for backend selection was no longer
available. 

## Anti-cheat

Before version 0.11.7.4xxx, EFT used their own in-house anti-cheat named
Saber, better known as `SClient.dll`. It was only used during online raids and
to detect cheat program signatures. It didn't have to be patched out for
running EFT offline.

Starting version 0.11.7.4xxx, EFT migrated to using BattlEye. It can detect
dll injection from BepInEx and other sources, but not from NLog targets. It can
also detect deobfuscation of `Assembly-CSharp.dll`. It is required to be
disabled for running EFT offline.

In all versions of the game, the sole function with the name `RunValidation` is
responsible for all anti-cheat checks that happen in offline raids and in the
main menu.

## File integrity

Up until 0.12.10, EFT used launcher-side integrity validation by comparing the
hashes inside `ConsistencyInfo` with the game files. Starting 0.12.11, the
client also validates file integrity on startup using the same method.

The validation check is split in two parts: the general files and game assets.
For running tarkov offline with deobfuscation, you can get away with disabling
the general files scan. If you want to mod the game bundles, the latter scan
must also be disabled.

## Comminucation

### Server

#### Protocol

Before version 0.12.0, EFT used HTTP for comminucation. Starting 0.12.0, it
uses HTTPS.

#### Request counting

Eft uses an internal request counter to keep track of the amount of requests
made. The counter starts at 0, and increments for each request made by 1. It
resets once the client logs out from the server or is disconnected for
inactivity.

The client is responsible for keeping track of the request counter. If the
counter between the client and server mismatches, the server will ban the user
for request botting.

For example:

```
[0] <starting point>
[1] https://prod.escapefromtarkov.com/client/game/start 
[2] https://prod.escapefromtarkov.com/client/menu/locale/en 
[3] https://prod.escapefromtarkov.com/client/game/version/validate
```

#### Compression

In every version of EFT, `zlib` (RFC1950) is used for (de)compression of
requests and responses. The client nor server sets the correct headers for
this. Including the correct headers results in automatic decompression by
Unity, which in turn breaks their request handler as it attempts to decompress
as well.

Special care must be taken when using EFT's
`componentace.compression.libs.zlib` library. `SimpleZlib`'s implementation
using `System.Buffer` can corrupt your data when two files are (de)compressed
at the same time in a multi-threaded scenario.

#### Response body

Every response made over HTTP by EFT's server has the following body:

```jsonc
{
    "err": 0,           // uint
    "errcode": null,    // string?
    "data": null,       // T?
    "crc": 0            // uint
}
```

The following properties are never stripped:

- `err`: The error code. Default to `0` (no error), otherwise an `uint`
  (example: `1293`)
- `errcode`: The message on error. Default to `null` (no error), otherwise a
  `string` (example: `"File integrity failed"`).
- `data`: The actual data of the response. Default to `null` or `[]` depending
   on `data`'s type (`JObject` or `JArray`), otherwise any value (`T`).

The following property might be stripped:

- `crc`: The CRC32 hash of `data`'s value. Defaults to `0` (not cached),
  otherwise an `uint` (example: `392282128`)

### Client

#### Caching

The client is capable of caching responses from various requests using ETags.
The client determines which requests need to be cached. It uses a `CRC32` hash
as `uint` type to hash the contents of the cache files.

The cache files are stored in `<gamedir>/cache/`. The filenames of the cached
data is the response path (example: `/client/items`) converted to `md5`.

The contents of the cached data is compressed using `zlib` (RFC1950) and
encrypted with an XOR key (`<byte> ^= 13`).

When the client has no cached file for the request, it will the header

```
"If-None-Match": 0
```

When the client has a cached file for the request, it will send the following
header:

```
"If-None-Match": 392282128 // hash
```

If the crc hash matches with the server's value, then response code `304` is
send back.

The client will use it's cached value.

If the crc hash doesn't match, then the following response body is send back:

```jsonc
{
    "err": 0,    
    "errcode": null,
    "data": {},         // the JObject / JArray with the server data
    "crc": 392282128    // the new hash
}
```

The client updates the content of the cache or creates a new file when
appropiate.

#### Default request headers

**Item**            | **Value type** | **Example value**
------------------- | -------------- | ---------------------------
`App-Version`       | `string`       | `EFT Client 0.13.5.0.25793`
`Cookie`            | `string`       | `PHPSESSID=409154`
`GClient-RequestId` | `uint`         | `2`

### Servers

The client makes requests to the following endpoints:

```
https://prod.escapefromtarkov.com
https://trading.escapefromtarkov.com
https://ragfair.escapefromtarkov.com
wss://prod.escapefromtarkov.com/sws
```

#### Startup

Depending on whenever the account is wiped or not, the client gets a different
request order.

When the game starts and your account's profile is established, the following
requests are made:

```
/client/game/start 
/client/menu/locale/en 
/client/game/version/validate
/client/languages
/client/game/config
/client/items
/client/customization
/client/globals
/client/trading/api/traderSettings
/client/settings
/client/game/profile/list
/client/game/profile/select
/client/profile/status
/client/weather
/client/locale/en
/client/locations
/client/handbook/templates
/client/hideout/areas
/client/hideout/qte/list
/client/hideout/settings
/client/hideout/production/recipes
/client/hideout/production/scavcase/recipes
/client/handbook/builds/my/list
/client/notifier/channel/create
/client/friend/list
/client/mail/dialog/list  
/client/friend/request/list/inbox 
/client/friend/request/list/outbox
/client/trading/customization/storage
/client/server/list
/client/match/group/current
/client/quest/list
/client/repeatalbeQuests/activityPeriods
```

When the account's profile doesn't exist on the server, the following requests
are made:

```
/client/game/start
/client/menu/locale/en
/client/game/version/validate
/client/languages
/client/game/config
/client/items
/client/customization
/client/globals
/client/trading/api/traderSettings
/client/settings
/client/game/profile/list
/client/locale/en
/client/account/customization
/client/locale/en
/client/locale/ru
/client/locale/ge
/client/locale/fr
/client/game/profile/nickname/reserved
/client/game/profile/nickname/validate
/client/game/profile/create
/client/game/profile/list
/client/game/profile/select
/client/profile/status
/client/weather
/client/locations
/client/handbook/templates
/client/hideout/areas
/client/hideout/qte/list
/client/hideout/settings
/client/hideout/production/recipes
/client/hideout/production/scavcase/recipes
/client/handbook/builds/my/list
/client/notifier/channel/create
/client/friend/list
/client/mail/dialog/list
/client/friend/request/list/inbox
/client/friend/request/list/outbox
/client/trading/customization/storage
/client/server/list
/client/match/group/current
/client/quest/list
/client/repeatalbeQuests/activityPeriods
```

If the account's profile doesn't exist on the server, the server returns the
following response for `/client/game/profile/list`:

```json
{
    "err": 0,
    "errcode": null,
    "data": []
}
```

> Fun fact: it took two years (2019-2021) to figure out how profile wiping
> worked. Previously it was assumed that Profile.Info.isWiped was responsible
> for determinating whenever to show character creation on startup. This proved
> to be false.

Both `/client/menu/locale/en` and `/client/locale/en` can also be another
language, depending on which language is set in the game's local settings file.

#### Exit

When the player exits the game through the exit button in the main menu, the
following request is made:

```
/client/game/logout
```

#### Synchronization

To synchronize data between the client and server, the client makes the
following request periodically:

```
/client/game/profile/items/moving
/client/profile/status
```

#### Ping

In order to signal that the client is still active, it periodically sends the
following request:

```
/client/game/keepalive
```

it also sends a notifier ping request over websocket.

## Internals

`AccountId` (also known as `AID` or `MongoId`) is an `uint` and used for:

- `-token` launch argument
- the value of `PHPSESSID` in the HTTP header's cookie
- profile selection

```
2023-08-12 22:38:44.036 +02:00|0.13.5.0.25793|Info|application|SelectProfile ProfileId:5a452fc902153a1e7403f888 AccountId:409154 
```

## Appendix. A: Request function map

**Server** | **Client function**                | **Path**
---------- | ---------------------------------- | -------------------------------------------------
Main       | `AcceptGroupInvite`                | `/client/match/group/invite/accept`
Main       | `CancelAllInvites`                 | `/client/match/group/invite/cancel-all`
Main       | `CancelInvite`                     | `/client/match/group/invite/cancel`
Main       | `ChangeNickname`                   | `/client/game/profile/nickname/change`
Main       | `ChangeVoice`                      | `/client/game/profile/voice/change`
Main       | `CreateNotifierChannel`            | `/client/notifier/channel/create`
Main       | `CreateProfile`                    | `/client/game/profile/create`
Main       | `DeclineGroupInvite`               | `/client/match/group/invite/decline`
Main       | `DisbandRaidGroup`                 | `/client/match/group/delete`
Main       | `ExitFrommatchmakerGroupMenu`      | `/client/match/group/exit_from_menu`
Main       | `FindAccountByNickname`            | `/client/game/profile/search`
Main       | `GetAvailableAccountCustomization` | `/client/account/customization`
Main       | `GetChatServer`                    | `/client/chatServer/list`
Main       | `GetClientSettingsConfig`          | `/client/settings`
Main       | `GetCustomization`                 | `/client/customization`
Main       | `GetGroupStatus`                   | `/client/match/group/current`
Main       | `GetLocalization`                  | `/client/locale/<localeId>`
Main       | `GetMetricsConfig`                 | `/client/getMetricsConfig`
Main       | `GetRaidReadyStatus`               | `/client/match/group/status`
Main       | `GetReservedNickname`              | `/client/game/profile/nickname/reserved`
Main       | `GetTemplates`                     | `/client/items`
Main       | `IsMatchingAvailable`              | `/client/match/available`
Main       | `LeaveMatchmakerGroup`             | `/client/match/group/leave`
Main       | `LoadBots`                         | `/client/game/bot/generate`
Main       | `LoadTextureMain`                  | Fully dynamic
Main       | `RemovePlayerFromGroup`            | `/client/match/group/player/remove`
Main       | `RequestHandbookInfo`              | `/client/handbook/templates`
Main       | `SendClientProfileSettings`        | `/client/profile/settings`
Main       | `SendGroupInvite`                  | `/client/match/group/invite/send`
Main       | `SendMetrics`                      | `/client/putMetrics`
Main       | `SetNotReadyRaidStatus`            | `/client/match/raid/not-ready`
Main       | `SetReadyRaidStatus`               | `/client/match/raid/ready`
Main       | `StartKeepAliveCoroutine`          | `/client/game/keepalive`
Main       | `StartLookingForGroup`             | `/client/match/group/looking/start`
Main       | `StopLookingForGroup`              | `/client/match/group/looking/stop`
Main       | `TransferGroupLeadership`          | `/client/match/group/transfer`
Main       | `ValidateNickname`                 | `/client/game/profile/nickname/validate`
Main       | `CheckVersion`                     | `/client/checkVersion`
Main       | `FinishScavSession`                | `/client/game/profile/savage/regenerate`
Main       | `GetAreasQte`                      | `/client/hideout/qte/list`
Main       | `GetAreaTemplatesUnparsed`         | `/client/hideout/areas`
Main       | `GetDailyQuests`                   | `/client/repeatalbeQuests/activityPeriods`
Main       | `GetGlobalConfig`                  | `/client/globals`
Main       | `GetHideoutSettings`               | `/client/hideout/settings`
Main       | `GetInsurancePrice`                | `/client/insurance/items/list/cost`
Main       | `GetLevelSettings`                 | `/client/locations`
Main       | `GetProductionRecipes`             | `/client/hideout/production/recipes`
Main       | `GetProfiles`                      | `/client/game/profile/list`
Main       | `GetScavRecipes`                   | `/client/hideout/production/scavcase/recipes`
main       | `GetSupplyData`                    | `/client/items/prices/<traderId>`
Main       | `GetWeatherAndTime`                | `/client/weather`
Main       | `LoadLocationLoot`                 | `/client/location/getLocalloot`
Main       | `LoadTextureRagfair`               | Fully dynamic
Main       | `Logout`                           | `/client/game/logout`
Main       | `OfflineRaidEnded`                 | `/client/match/offline/end`
Main       | `RagfairGetPrices`                 | `/client/items/prices`
Main       | `RagfairReportOffer`               | `/client/reports/ragfair/send`
Main       | `ReceiveCoopRaidSettings`          | `/client/raid/configuration-by-profile`
Main       | `ReceiveInsurancePrices`           | `/client/insurance/items/cost`
Main       | `RefreshPings`                     | `/client/server/list`
Main       | `RequestBuilds`                    | `/client/handbook/builds/my/list`
Main       | `RequestQuestsTemplates`           | `/client/quest/list`
Main       | `SendRaidSettings`                 | `/client/raid/configuration`
Main       | `SetmainProfile`                   | `/client/game/profile/select`
Main       | `SendReport`                       | `/client/reports/abuse/send`
Messaging  | `AcceptAllFriendRequests`          | `/client/friend/request/accept-all`
Messaging  | `AcceptFriendRequest`              | `/client/friend/request/accep`
Messaging  | `AllAttachmentsFromDialog`         | `/client/mail/dialog/getAllAttachments`
Messaging  | `CancelFriendRequest`              | `/client/friend/request/cancel`
Messaging  | `ChatCreateGroupDialog`            | `/client/mail/dialog/group/create`
Messaging  | `ChatDeleteAllMessages`            | `/client/mail/dialog/clear`
Messaging  | `ChatGetDialogList`                | `/client/mail/dialog/list`
Messaging  | `ChatGetDialogMessages`            | `/client/mail/dialog/view`
Messaging  | `ChatReadDialogues`                | `/client/mail/dialog/read`
Messaging  | `ChatSendMessage`                  | `/client/mail/msg/send`
Messaging  | `DeclineFriendRequest`             | `/client/friend/request/decline`
Messaging  | `DeleteDialog`                     | `/client/mail/dialog/remove`
Messaging  | `GetDialogInformation`             | `/client/mail/dialog/info`
Messaging  | `GetFriendList`                    | `/client/friend/list`
Messaging  | `GetInputFriendRequest`            | `/client/friend/request/list/inbox`
Messaging  | `GetOutputFriendRequest`           | `/client/friend/request/list/outbox`
Messaging  | `InviteToDialog`                   | `/client/mail/dialog/group/users/add`
Messaging  | `LeaveChatDialog`                  | `/client/mail/dialog/group/leave`
Messaging  | `MutePlayer`                       | `/client/friend/ignore/set`
Messaging  | `PinDialog`                        | `/client/mail/dialog/pin`
Messaging  | `RemoveFromDialog`                 | `/client/mail/dialog/group/users/remove`
Messaging  | `RemoveFromFriendList`             | `/client/friend/delete`
Messaging  | `SendFriendRequest`                | `/client/friend/request/send`
Messaging  | `TransferChatLeadership`           | `/client/mail/dialog/group/owner/change`
Messaging  | `UnmutePlayer`                     | `/client/friend/ignore/remove`
Messaging  | `UnpinDialog`                      | `/client/mail/dialog/unpin`
Trading    | `GetAvailableSuits`                | `/client/trading/customization/storage`
Trading    | `GetOffers`                        | `/client/trading/customization/<traderId>/offers`
Trading    | `GetTraderAssort`                  | `/client/trading/api/getTraderAssort/<traderId>`
Trading    | `LoadAvatar`                       | Fully dynamic
Ragfair    | `FindById`                         | `/client/ragfair/offer/findbyid`
Ragfair    | `GetMarketPrice`                   | `/client/ragfair/itemMarketPrice`
Ragfair    | `GetOffers`                        | `/client/ragfair/find`
Lobby      | `EstablishWSConnection`            | `/<token>`

Notes:

- `EstablishWSConnection`: Websocket token is set to `<token>` and `preAuth` is
  `true`

## Appendix B. Unused requests inside the client

**Server** | **Client function**                | **Path**
---------- | ---------------------------------- | ----------------------------
Main       | `GetCaptcha`                       | `/client/captcha/get`
Main       | `ValidateCaptcha`                  | `/client/captcha/validate`
Main       | `ReportNickname`                   | `/client/reports/lobby/send`
Main       | `IssueWSToken`                     | `/client/game/token/issue`
Trading    | `GetGlobalConfig`                  | `/client/globals`
