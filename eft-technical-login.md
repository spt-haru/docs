# Logging in

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

The config in the launch arguments is in the following format:

```jsonc
{
    "backendUrl": "", // http(s) endpoint
    "version": ""     // access (either "live" or "dev")
}
```

Starting 0.12.0, using `client.config.json` for backend selection was no longer
available. 
