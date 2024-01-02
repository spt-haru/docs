# Packet sniffing

How to obtain server data from the client without using external tools like Fiddler.

## Requirements

- de4dot
- dnspy

## Deobfuscation

1. dnspy > file > open > `Assembly-CSharp.dll`
2. dnspy > file > export to project > export
3. vscode > file > open folder > the exported source folder
4. vscode > search > `AppDomain.CurrentDomain.GetData`, the method looks like this:

```cmd
internal sealed class \uED30
{
	// Token: 0x0600F6FF RID: 63231 RVA: 0x0049D660 File Offset: 0x0049B860
	public static string \uE000(int \uE001\uE5B7)
	{
		return (string)((Hashtable)AppDomain.CurrentDomain.GetData(\uED30.\uE002))[\uE001\uE5B7];
	}
}
```

5. de4dot > clean the assembly (change `0x0600F6FF` to the token)

```cmd
de4dot-x64.exe --un-name "!^<>[a-z0-9]$&!^<>[a-z0-9]__.*$&![A-Z][A-Z]\$<>.*$&^[a-zA-Z_<{$][a-zA-Z_0-9<>{}$.`-]*$" "Assembly-CSharp.dll" --strtyp delegate --strtok 0x0600F6FF
pause
```

### Fix ResolutionScope error

1. copy-paste `Assembly-CSharp-cleaned.dll` into `<gamedir>/EscapeFromTarkov_Data/Managed/`
2. dnspy > file > open > `Assembly-CSharp-cleaned.dll`
3. dnspy > file > save module > ok

## Modifications

These are done in dnspy on `Assembly-CSharp-cleaned.dll`

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
