# xmlParser

A small static C++ library for parsing XML, used across several LBNL projects.

## Requirements

- C++20 compatible compiler (`g++` 11+, `clang++` 14+, MSVC 19.30+)
- CMake 3.8+ (3.21+ if you want to use the shipped CMake presets)

## Consuming xmlParser

### Using FetchContent in CMake (recommended)

```cmake
include(FetchContent)
FetchContent_Declare(
    xmlParser
    GIT_REPOSITORY https://github.com/LBNL-ETA/XMLParser.git
    GIT_TAG v1.0.5
)
FetchContent_MakeAvailable(xmlParser)

target_link_libraries(MyTarget PRIVATE xmlParser)
```

Update `GIT_TAG` to the desired release tag.

### Using `add_subdirectory`

```cmake
add_subdirectory(path/to/XMLParser)
target_link_libraries(MyTarget PRIVATE xmlParser)
```

## Building (developers)

xmlParser has no external dependencies, so a developer's loop is just configure + build. CMake presets are provided for convenience and to keep the workflow consistent with the other LBNL repos.

### Presets

`CMakePresets.json` ships four visible configure presets, plus two hidden inheritance bases:

| Preset | When to use it |
|---|---|
| `default-debug` / `default-release` | Standard configure on any platform. Picks the system default compiler (MSVC on Windows, system `cc`/`c++` on Linux/macOS). |
| `local-debug` / `local-release` | Same as `default-*` today (xmlParser has no `FetchContent` deps yet). Kept in the structure so personal compiler presets can `inherits: "local"` and the file stays forward-compatible if a dep is ever added. |

Examples:

```
cmake --preset default-release
cmake --build build/default-release --parallel
```

Each preset writes into its own subdirectory under `build/` so Debug and Release artifacts don't clobber each other.

#### Per-machine compiler presets (`CMakeUserPresets.json`)

To use a specific compiler (`vs2022-release`, `gcc-13-debug`, `clang-18-release`, etc.), each developer maintains their own `CMakeUserPresets.json` next to `CMakePresets.json`. It is gitignored, read automatically by CMake (and CLion, VS Code, etc.), and stays on the developer's machine.

Personal presets `inherit` from one of the shipped presets (usually `local`) and override whatever they want. A complete realistic example — building with WSL Clang on a Windows machine, with CLion 2023.2+ routed through the WSL toolchain automatically:

```json
{
    "version": 6,
    "configurePresets": [
        {
            "name": "clang-release",
            "displayName": "clang (Release)",
            "inherits": "local",
            "generator": "Ninja",
            "binaryDir": "${sourceDir}/build/clang-release",
            "cacheVariables": {
                "CMAKE_C_COMPILER":   "clang",
                "CMAKE_CXX_COMPILER": "clang++",
                "CMAKE_BUILD_TYPE":   "Release"
            },
            "vendor": {
                "jetbrains.com/clion": {
                    "toolchain": "WSL"
                }
            }
        }
    ]
}
```

A few things going on in that one preset:

- `"inherits": "local"` → picks up sibling-repo overrides (when present) and the rest of the framework setup.
- Bare compiler names (`clang`, `clang++`) rather than `/usr/bin/clang` → portable to any machine that has that toolchain on `PATH`. Use absolute paths only if the compiler isn't on `PATH` (e.g. `C:/Program Files/LLVM/bin/clang.exe` — forward slashes work in JSON, no escaping needed).
- `"vendor.jetbrains.com/clion.toolchain"` → tells CLion (2023.2+) which configured toolchain to route this preset through. Standard names are `WSL`, `Visual Studio`, `MinGW`; whatever you see in `Settings → Build, Execution, Deployment → Toolchains`. The hint is silently ignored if the name doesn't match — no configure-time error.

Add as many of those blocks as you have toolchains you want explicit presets for (one per compiler × build type). Each gets its own `binaryDir` so Debug and Release artifacts don't clobber each other.

Alternative if you don't want a personal preset at all: set `CC` and `CXX` environment variables in your shell rc (`~/.bashrc`, PowerShell profile) before invoking `cmake --preset default-release`. CMake picks them up.

### Manual configure (without presets)

```
cmake -B build
cmake --build build --config Release --parallel
```

### Clean rebuild

Delete the `build/` directory and re-run the configure and build commands above.

## License

See the [LICENSE](LICENSE) file.
