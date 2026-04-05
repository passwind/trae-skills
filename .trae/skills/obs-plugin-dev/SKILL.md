---
name: "obs-plugin-dev"
description: "OBS plugin development assistant. Invoke when the user wants to develop, build, or manage C++ code, Qt6 UI, and i18n for an OBS plugin."
---

# OBS Plugin Development Skill

This skill provides guidelines and conventions for developing an OBS plugin based on the `obs-plugintemplate`.

## Reference
Please refer to `https://github.com/sorayuki/obs-multi-rtmp` for development examples and patterns.

## Core Requirements
1. **Code Location**: All C++ code must be placed in a specified directory (configurable/specifiable, with a default value).
2. **Naming Convention**: All files and class names must start with a default prefix to remain consistent with the rest of the project.
3. **UI Integration**: The settings window must use a dock widget, loaded via the OBS dock menu or a specified menu.
4. **i18n**: The plugin must support internationalization (i18n).

## Development Conventions
1. **Language & Headers**: Use C++ for development. Header files must use the `.hpp` extension and include the `#pragma once` directive.
2. **Language**: All comments and logs must be in English.
3. **i18n Implementation**: UI text output must support i18n. Modify the corresponding language files under the `data/locale` directory.
4. **UI Components**: Implement UI components in separate classes (do not put multiple components in a single file). Uniformly place all UI component files in the `ui` directory.
5. **Qt6 Signals/Slots**: If Qt6 signal support is needed, directly include the MOC file in the component's `.cpp` file using the format `#include "moc_<class_cpp_file_name>.cpp"`. The CMake project will automatically generate this file. 
   *Example*: In `ttoutput-config-dialog.cpp`, add `#include "moc_ttoutput-config-dialog.cpp"`.
6. **CMakeLists.txt**: Avoid modifying `CMakeLists.txt` unless you are updating the `src` file list.
7. **File Endings**: All C/C++ files must end with an empty newline to prevent compilation errors.

## Build Instructions
This project is based on the OBS plugin template. Qt6 and the front API are enabled.

- **Configure**: Prefer `cmake --preset macos-user` when `CMakeUserPresets.json` exists; otherwise use `cmake --preset macos`.
- **Build**: Prefer `cmake --build --preset macos-user`; otherwise use `cmake --build --preset macos`.
- **Output**: Build results are located in the `build_macos` directory.
- **Restrictions**: 
  - DO NOT use generic `cmake` build commands. 
  - Xcode build debugging can be used if absolutely necessary.
  - Do NOT use `clang` for compilation processing due to include path issues.
- **Dependencies**: All OBS, Qt, and related dependency source files are located in the `.deps` directory. Check the CMake cache if necessary.

Notes:

- The repository's `CMakePresets.json` defines the `macos` preset with Xcode generator and `RelWithDebInfo` configuration. If `Debug` is needed, prefer a local preset in `CMakeUserPresets.json` rather than changing shared presets.

## Convention: "cmake"

Interpretation:

- When the user says "cmake", they want a **configure** step using the existing preset (default: `macos`).
- The configure step may include a configurable set of `-D<KEY>=<VALUE>` definitions.
- Treat any API keys/secrets as sensitive: do not hardcode them in code, do not commit them, and avoid echoing them back in plain text.

Preferred configuration (per-project, non-committed): CMakeUserPresets.json

- Use a `CMakeUserPresets.json` in the project root to store per-project overrides locally.
- Keep it out of version control (do not commit).
- Create a configure preset that inherits the template preset and adds `cacheVariables`.

Example structure (placeholders only):

```json
{
  "version": 5,
  "configurePresets": [
    {
      "name": "macos-local",
      "inherits": "macos",
      "cacheVariables": {
        "YOUTUBE_API_CLIENT_ID": "$env{YOUTUBE_API_CLIENT_ID}",
        "YOUTUBE_API_CLIENT_SECRET": "$env{YOUTUBE_API_CLIENT_SECRET}",
        "TWITCH_API_CLIENT_ID": "$env{TWITCH_API_CLIENT_ID}",
        "ONESEVENLIVE_API_URL": "$env{ONESEVENLIVE_API_URL}",
        "CMAKE_PROJECT_VERSION": "$env{CMAKE_PROJECT_VERSION}"
      }
    }
  ]
}
```

Then run:

- `cmake --preset macos-local`

Alternative (ad-hoc): pass `-D` flags on the command line

- Use environment variables as the source of truth, then expand them in the command.
- Quote values to handle special characters safely.

Command template (placeholders only):

- `cmake --preset macos -DYOUTUBE_API_CLIENT_ID="$YOUTUBE_API_CLIENT_ID" -DYOUTUBE_API_CLIENT_SECRET="$YOUTUBE_API_CLIENT_SECRET" -DTWITCH_API_CLIENT_ID="$TWITCH_API_CLIENT_ID" -DONESEVENLIVE_API_URL="$ONESEVENLIVE_API_URL" -DCMAKE_PROJECT_VERSION="$CMAKE_PROJECT_VERSION"`

Behavior:

1. If `CMakeUserPresets.json` exists and defines a usable preset (e.g., `macos-user`), prefer it.
2. Otherwise, use `cmake --preset macos` and append configured `-D` flags.
3. For missing required values, ask the user to provide them or to export them as environment variables.

## Git Workflow
After a successful build, commit the changes to Git:
- Use an English commit message.
- Keep the commit message under 20 words.
- DO NOT push the commit.
