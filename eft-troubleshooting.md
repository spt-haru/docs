# T

Working with EFT can be quite challenging due to lack of available debugging
tools. Couple of things you can do:

1. Enable NLog loggers
2. Read `traces.log`
3. Read `player.log`

## NLog loggers

In `<gamedir>/NLog/NLog.config` there are many loggers under the `<rules>`
section. Set these all to `minlevel="Trace"` for full output. Provides
sometimes more detailed logs.

## Traces.log

In `<gamedir>/Logs/<datetime>/traces.log` you'll find almost everything logged
in the game, mostly in order of occurance. This is the first place to check if
something went wrong.

## Player.log

If EFT is behaving weird and it cannot be explained from `traces.log` or if
there are no logs at all, open up 
`C:/Users/<usr>/Appdata/LocalLow/Battlestate Games/EscapeFromTarkov/Player.log`
and see what's going on there.

## Common issues

> The game closes as soon as it starts

Integrity verification kicked in and aborted booting. You need to disable it
(through a harmony patch), see `FilesChecker.dll`.

> The game softlocks on startup past the first loading screen, doesn't show
> profile data loading.

Can be multiple things:

1. Server it's trying to connect to cannot be reached (start the server)
2. Wrong startup parameters
(`-token=<uid> -config={"BackendUrl":"<server address>","Version":"live"`)
3. An exception was thrown (see `traces.log` or `player.log`)

> Profile data loads but aborts and shows a verification error message

That's BattlEye, you need to disable it with a harmony patch (look for
`RunValidation` method, make sure `this.Succeed` is true).
