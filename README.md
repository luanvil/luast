<div align="center">

<img src="assets/luast.svg" width="64" height="64" alt="luast logo">

# luast

Build standalone Lua executables with cross-compilation support.

[![License: MIT](https://img.shields.io/badge/license-MIT-000080)](LICENSE)

</div>

## Features

<table>
<tr>
<td width="50%">

**Cross-Compilation**

Build for Linux, macOS, and Windows from any platform. Powered by Zig.

</td>
<td width="50%">

**Standalone Executables**

Single-file executables with no runtime dependencies. Linux uses musl libc.

</td>
</tr>
<tr>
<td width="50%">

**C Library Support**

Built-in support for luafilesystem, lpeg, lua-cjson, lsqlite3, luasocket.

</td>
<td width="50%">

**Asset Embedding**

Embed data files and directories directly into the binary.

</td>
</tr>
</table>

## Quick Start

```bash
# Build for current platform
luast myapp

# Build for multiple targets
luast -t linux-x86_64 -t darwin-arm64 -t windows-x86_64 myapp

# Standalone mode (no rockspec)
luast -m app.lua myapp
```

## Usage

```
luast [options] [modules...] [output-name]

Options:
    -h, --help      Show help message
    -q, --quiet     Only show errors and warnings
    -v, --verbose   Show detailed output
    -m, --main      Main entry point (standalone mode)
    -c, --clib      C library dependency (can be repeated)
    -e, --embed     Embed data file/directory (can be repeated)
    -r, --rockspec  Path to rockspec file (default: auto-detect)
    -t, --target    Target platform (default: native)
```

### Targets

| Target | Output |
|--------|--------|
| `linux-x86_64`, `linux-arm64` | Static ELF (musl) |
| `darwin-x86_64`, `darwin-arm64` | Mach-O |
| `windows-x86_64`, `windows-arm64` | PE32+ |

### Environment

| Variable | Description | Default |
|----------|-------------|---------|
| `BUILD_DIR` | Build directory | `.build` |
| `LUAST_CACHE` | Cache directory | `~/.cache/luast` |
| `LUA_VERSION` | Lua version | `5.4.8` |

## Standalone Mode

Build without a rockspec:

```bash
luast -m app.lua myapp
luast -m app.lua lib/ src/ myapp
luast -m app.lua -c lfs -c lpeg myapp
```

Include LuaRocks modules by path:

```bash
luarocks install --local inspect
luast -m app.lua ~/.luarocks/share/lua/5.4/inspect.lua myapp
```

## Embedding Files

```bash
luast -m app.lua -e templates/ -e config.json myapp
```

Access at runtime:

```lua
local embed = require("luast.embed")
local html = embed.read("templates/page.html")
```

## C Libraries

Built-in support:

| Library | Notes |
|---------|-------|
| luafilesystem | |
| lpeg | |
| lua-cjson | |
| lsqlite3complete | Includes SQLite3 |
| luasocket | Includes socket.lua, ltn12.lua, mime.lua |

### Custom Libraries

Add via `.luastrc`:

```lua
return {
    mylib = {
        url = "https://github.com/user/mylib.git",
        sources = { "src/mylib.c" },
    },
}
```

## GitHub Action

```yaml
- uses: owloops/luast@main
  with:
    output: myapp
    targets: linux-x86_64 linux-arm64 darwin-arm64 windows-x86_64
```

| Input | Description | Default |
|-------|-------------|---------|
| `output` | Binary name | Package name |
| `rockspec` | Rockspec path | Auto-detect |
| `targets` | Space-separated targets | Native |

## Requirements

- Linux or macOS
- Lua 5.1+
- git, curl or wget

Zig is downloaded automatically.

## Limitations

- Linux binaries use musl (not glibc)
- No automatic LuaRocks dependency resolution
- No OpenSSL/TLS support

## LuaJIT Support

Build with LuaJIT instead of Lua 5.4:

```bash
LUA_VERSION=jit luast myapp
```

Requires `make` and `gcc`/`clang` on the host for LuaJIT's bootstrap process.

## Development

```bash
luacheck luast      # Lint
stylua luast        # Format
```

## Credits

Based on [luastatic](https://github.com/ers35/luastatic) by ers35. Powered by [Zig](https://ziglang.org/).

## License

[MIT](LICENSE)
