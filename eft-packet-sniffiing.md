# Packet sniffing

Patches required to obtain server data from the client without using external tools like Fiddler.

## Requirements

- dnspy ([link](https://github.com/spt-haru/dnspy))
- `Assembly-CSharp-cleaned.dll` (produced by deobfuscation)

## Patches

These are done in dnspy on `Assembly-CSharp-cleaned.dll`.

### Save requests / responses

1. search for `backRequest`
2. modify `method_5`; insert this at the bottom, just before `return text2;`

```cs
var uri = new Uri(backRequest.MainURLFull);
var path = (System.IO.Directory.GetCurrentDirectory() + "\\HTTP_DATA\\").Replace("\\\\", "\\");
var file = uri.LocalPath.Replace('/', '.').Remove(0, 1);
var time = DateTime.Now.ToString("yyyy-MM-dd_HH-mm-ss");

if (System.IO.Directory.CreateDirectory(path).Exists)
{
    if (backRequest.Params != null)
    {
        System.IO.File.WriteAllText($@"{path}req.{file}_{time}.json", backRequest.Params.ToJson());
    }

    System.IO.File.WriteAllText($@"{path}resp.{file}_{time}.json", text2);
}
```

## BattlEye

1. search for `RunValidation`
2. replace `RunValidation`'s method body with this:

```cs
{
    this.Succeed = true;
}
```

## File integrity

1. search for `RunFilesChecking`
2. replace `RunFIlesChecking`'s method body with this:

```cs
{
}
```

## Injection

1. Rename `Assembly-CSharp-cleaned.dll` to `Assembly-CSharp.dll`.
2. Copy-paste `Assembly-CSharp.dll` into `EscapeFromTarkov_Data`, do not interact with the prompt yet
3. Start the game from Battlestate Games Launcher
4. As soon as the launcher closes, click "replace" in the prompt
