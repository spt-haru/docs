# Caching

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
send back. The client will use it's cached value.

If the crc hash doesn't match, then the following response body is send back:

```jsonc
{
    "err": 0,    
    "errcode": null,
    "data": {},         // the JObject / JArray with the server data
    "crc": 392282128    // the new hash
}
```

The client updates the content of the cache or creates a new file where
appropiate.
