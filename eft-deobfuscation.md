# Deobfuscating the client

## Requirements

- de4dot ([link](https://github.com/spt-haru/de4dot))
- dnspy ([link](https://github.com/spt-haru/dnspy))
- `Assembly-CSharp.dll` (from live EFT)

### Steps

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

5. Run the following (change `0x0600F6FF` to the token)

```cmd
de4dot.exe --un-name "!^<>[a-z0-9]$&!^<>[a-z0-9]__.*$&![A-Z][A-Z]\$<>.*$&^[a-zA-Z_<{$][a-zA-Z_0-9<>{}$.`-]*$" "Assembly-CSharp.dll" --strtyp delegate --strtok 0x0600F6FF
pause
```

### 2. Fix ResolutionScope error

1. copy-paste `Assembly-CSharp-cleaned.dll` into `<gamedir>/EscapeFromTarkov_Data/Managed/`
2. dnspy > file > open > `Assembly-CSharp-cleaned.dll`
3. dnspy > file > save module > ok
