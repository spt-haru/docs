# Embedded resource loading (resx)

Haru provides an embedded resource loading mechanism to allow for easy
integration and loading of assets (json, images, etc) located inside the
assembly.

## When to use resx

It's meant for (many) small files (below 8MB each), not for unity asset bundles
or larger textures. Use `VFS` (virtual file system) for those instead.

## Loading data

To load data from an assembly, get the assembly you want to load from and
register it to `Resx`. Then, use the `GetText()` (for `string`) or `GetData()`
(for `byte[]`) method to obtain the data.

```cs
using Haru.IO;

class Test
{
    public void Load()
    {
        // get the assembly to load from
        var assembly = GetType().Assembly;

        // initialize resource loader
        var resx = new Resx(assembly);

        // load <assembly>.embedded.test.txt
        var file = "test.txt";
        var text = resx.GetText(file);
        var data = resx.GetData(file);
    }
}
```

## Adding assets

In order to allow `Resx` to load your files, you'll need the following project
structure:

```
myproject/
  embedded/
    test.txt
  myproject.csproj
```

In addition, make sure your `myproject.csproj` includes an `EmbeddedResource`
reference to the file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <!-- ... -->

  <ItemGroup>
    <!-- set the file here to embed -->
    <EmbeddedResource Include="embedded/test.txt" />
  </ItemGroup>

</Project>

```

## Advanced

If you need the raw stream instead of `string` or `byte[]`, you can use the
`GetStream()` method instead. Keep in mind that you're responsible for
disposing the stream after use.

```cs
using Haru.Buffers;
using Haru.IO;

class Test
{
    private byte[] LoadResx(string file)
    {
        var assembly = GetType().Assembly;
        var resx = new Resx(assembly);

        // get the stream and dispose on completion
        using (var ms = MemoryStreamPool.Rent())
        {
            using (var rs = resx.GetStream(file))
            {
                rs.CopyTo(ms);
            }

            return ms.ToArray();
        }
    }

    public void Load()
    {
        // load <assembly>.embedded.test.txt
        var file = "test.txt";
        var data = LoadResx(file);
    }
}
```
