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

Built-in support for luafilesystem, lpeg, lua-cjson, lsqlite3, luasocket, luasec, luaossl.

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
    -R, --rock      LuaRocks package to include (can be repeated)
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

## LuaRocks Integration

Include installed LuaRocks packages with `-R`:

```bash
# Install a rock with dependencies
luarocks install inspect

# Build with all dependencies resolved automatically
luast -m app.lua -R inspect myapp
```

This discovers all Lua files from the rock and its dependencies, and automatically includes required C libraries.

Include multiple rocks:

```bash
luast -m app.lua -R inspect -R penlight myapp
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
| luafilesystem | File system operations |
| lpeg | Parsing expression grammars |
| lua-cjson | JSON encoding/decoding |
| lsqlite3complete | SQLite3 database (includes sqlite3.c) |
| luasocket | TCP/UDP sockets, HTTP client |
| luasec | SSL/TLS for luasocket (OpenSSL statically linked) |
| luaossl | OpenSSL bindings (OpenSSL statically linked) |

OpenSSL is cross-compiled and statically linked with optimized settings: legacy ciphers (RC4, IDEA, Blowfish, etc.), regional algorithms (SM2/3/4, GOST, Camellia, SEED), and unused features (compression, engines) are disabled. TLS 1.0-1.3, DTLS, and all modern ciphers (AES, ChaCha20) remain fully supported.

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
| `main` | Main entry point (standalone mode) | - |
| `rockspec` | Rockspec path | Auto-detect |
| `targets` | Space-separated targets | Native |
| `lua-version` | Lua version (`5.4.8` or `jit`) | `5.4.8` |
| `clibs` | Space-separated C libraries | - |
| `rocks` | Space-separated LuaRocks packages | - |
| `embeds` | Space-separated files to embed | - |
| `quiet` | Only show errors/warnings | `false` |

## Requirements

**Always required:**

- Linux or macOS host
- Lua 5.1+ (for running luast itself)
- git, tar
- curl or wget

**For OpenSSL-dependent libraries** (luasec, luaossl):

- Perl modules for OpenSSL's Configure script:
  - Fedora/RHEL: `sudo dnf install perl-FindBin perl-IPC-Cmd perl-File-Compare perl-File-Copy perl-Pod-Html`
  - Debian/Ubuntu: `sudo apt install perl` (modules included by default)
- OpenSSL headers: `openssl-devel` (Fedora/RHEL) or `libssl-dev` (Debian/Ubuntu)

**For LuaJIT builds** (`LUA_VERSION=jit luast myapp`):

- make
- gcc or clang

Zig is downloaded automatically.

## Development

```bash
luacheck luast      # Lint
stylua luast        # Format
```

## Credits

Based on [luastatic](https://github.com/ers35/luastatic) by ers35. C embedding approach inspired by [LuaRocks](https://github.com/luarocks/luarocks) binary builder. Powered by [Zig](https://ziglang.org/).

## License

[MIT](LICENSE)
