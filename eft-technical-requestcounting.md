# Request counting

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

The current counter value is located in the request's `GClient-RequestId`
header.
