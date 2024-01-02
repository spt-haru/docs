# Frequently asked questions

> Q: Why does the solution need both dotnet 4.7.1 and 8.0.x?

A: Because EFT bundles a mono runtime (supporting dotnet 4.7.1, C# 7) and the
   server itself is using a more modern runtime than what EFT provides.

> Q: Why is C# 6.0 specified for dotnet 8.0.x dotnet standard 2.0 projects?

A: To keep the language version consistent across projects. This helps alot
   with code refactoring and standardizing pratices within the projects. The
   lowest allowed version is used to ease transition to newer versions when
   required.

> Q: When do I use what dotnet version?

A: A quick overview.

- Client (or related): framework 4.7.1
- Server (or related): 8.0.x
- Shared libraries: standard 2.0

> Q: Why won't you add support for my IDE (VS2022, Rider, etc)?

A: Because I only use Visual Studio Code. I cannot make garuantees that it
   will remain functional when I don't use nor test.
