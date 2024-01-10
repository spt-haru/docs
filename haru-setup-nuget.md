# Setting up the package stream

Haru uses github's organization package stream to host it's own packages.
The upside is that Haru doesn't pollute nuget's package stream with EFT related
packages. The downside is that you need a github public access token (PAT) to
pull the packages.

### Generating a PAT

1. Github account > Settings > Developer settings
2. Public access tokens > Tokens (Classic) > Generate new token (classic)
3. Select the correct info
  - Set note to "read github packages"
  - Set exipiry date to whatever you want, I set it to "never"
  - Set scope to `read:packages`
4. Generate token

If you are the maintainer of this project, set scope to `write:packages` and
update the organization secret `TOKEN_PACKAGES` accordingly.

Make sure to save the token produced somewhere, you only get to see it once!

### Add package stream

Run the following inside a console:

- Replace `NAME` with your github username
- Replace `PAT` with your PAT

```sh
dotnet nuget add source "https://nuget.pkg.github.com/spt-haru/index.json" --name "spt-haru" --username "NAME" --password "PAT"
```

...and that's it! If everything works correctly, you can now use
`dotnet restore` to obtain the packages from the spt-haru package stream.
