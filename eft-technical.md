# EFT Server documentation

**TODO: SPLIT THIS UP INTO MULTIPLE FILES.**

This is a rough documentation of EFT's server endpoints, data and how it's
processed. This includes historical data and background information relevant
to make EFT's "offline raid" actually function offline.

## Comminucation

### Server

#### Protocol

Before version 0.12.0, EFT used HTTP for comminucation. Starting 0.12.0, it
uses HTTPS.

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
