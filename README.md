# Lumin - Live Wallpaper SDK 
![Logo](images/Logo.png)

This library allows users to create dynamic wallpapers on Windows, macOS and Linux

## Features

- Supports Windows 11 24H2 and prior!
- Provides mouse input replacements for interactive desktops
- Supports Multi Monitor Setups and is DPI aware.
- Doesn't Render if Wallpaper or Monitor is occluded

## Getting Started

### Installation
Simply add the library to your CMake Config
```md
# Specify branch or version
set(LWP_GIT_TAG main CACHE STRING "Git tag/branch for LuminWallpaper")

# Fetch LuminWallpaper
FetchContent_Declare(
  lumin
  GIT_REPOSITORY https://github.com/jensroth-git/LuminWallpaper.git
  GIT_TAG ${LWP_GIT_TAG}
)
FetchContent_MakeAvailable(lumin)

# Add to target
target_link_libraries(${PROJECT_NAME} PRIVATE lumin)
```

### raylib integration
TODO...

### Unity integration
TODO...

### Unreal integration
TODO...

## Future Plans
Currently, there are no replacements for keyboard input, which may be added in the future.

## License

This project is licensed under the MIT License.

