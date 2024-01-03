# File integrity scanning

Up until 0.12.10, EFT used launcher-side integrity validation by comparing the
hashes inside `ConsistencyInfo` with the game files. Starting 0.12.11, the
client also validates file integrity on startup using the same method.

The validation check is split in two parts: the general files and game assets.
For running tarkov offline with deobfuscation, you can get away with disabling
the general files scan. If you want to mod the game bundles, the latter scan
must also be disabled.

`FilesChecker.dll` contains all the logic to enable EFT to check if file sizes
are still correct, if the required files exist at all, and for some if the
checksum matches.

the file `ConsistencyInfo` is ignored by the client and exclusively used by the
`BsgLauncher`. Instead, the class `ConsistencyMetadataProvider` contains a list
with all the metadata.

The "critical" check differs between the two;

- `BsgLauncher` check the MD5 hash of the file
- `FilesChecker.dll` combines byte values (logic located inside
  `ConsistencyController`)

Priority modes determine which checks run:

**Priority**        | **Checks**
------------------- | --------------------------------------------------------
normal              | if the file exists, file size match
critical (client)   | if the file exists, file size match, file checksum match
critical (launcher) | if the file exists, file size match, file hash match
