# Startup requests

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

> Fun fact: it took a year (2019-2020) to figure out how profile wiping worked.
> Previously it was assumed that Profile.Info.isWiped was responsible for
> determinating whenever to show character creation on startup. This proved
> to be false.

Both `/client/menu/locale/en` and `/client/locale/en` can also be another
language, depending on which language is set in the game's local settings file.
