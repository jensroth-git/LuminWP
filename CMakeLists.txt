cmake_minimum_required(VERSION 3.20)

project(lumin
  VERSION 1.0.0
  DESCRIPTION "Universal Live Wallpaper SDK"
  LANGUAGES C CXX)

# Only turn on ObjC/ObjCXX on Apple, so Windows stays happy
if(APPLE)
  enable_language(OBJC)
  enable_language(OBJCXX)
endif()

# Compat for PROJECT_IS_TOP_LEVEL on older CMake
if(NOT DEFINED PROJECT_IS_TOP_LEVEL)
  if(CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
    set(PROJECT_IS_TOP_LEVEL ON)
  else()
    set(PROJECT_IS_TOP_LEVEL OFF)
  endif()
endif()

include(GNUInstallDirs)
option(LWP_BUILD_EXAMPLES "Build examples" ${PROJECT_IS_TOP_LEVEL})

if(UNIX AND NOT APPLE)
  option(LWP_USE_GLFW "Use GLFW-native handle path on Linux (for raylib/GLFW engines)" ON)
endif()

# Library target.
add_library(${PROJECT_NAME} STATIC
  $<$<PLATFORM_ID:Windows>:src/platform/windows/desktop_integration_windows.cpp>
  $<$<PLATFORM_ID:Darwin>:src/platform/macos/desktop_integration_macos.mm>
  $<$<PLATFORM_ID:Linux>:src/platform/linux/desktop_integration_linux.cpp>
)

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)

target_include_directories(${PROJECT_NAME} PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

set_property(TARGET ${PROJECT_NAME} PROPERTY POSITION_INDEPENDENT_CODE ON)

# Platform links
if(WIN32)
  target_compile_definitions(${PROJECT_NAME} PUBLIC LWP_PLATFORM_WINDOWS)
  target_link_libraries(${PROJECT_NAME} PUBLIC user32 gdi32 shcore shlwapi dwmapi)
elseif(APPLE)
  target_compile_definitions(${PROJECT_NAME} PUBLIC LWP_PLATFORM_MACOS)

  find_library(COCOA_LIBRARY Cocoa)
  find_library(IOKIT_LIBRARY IOKit)
  find_library(COREVIDEO_LIBRARY CoreVideo)
  find_library(OPENGL_LIBRARY OpenGL)
  find_library(CARBON_LIBRARY Carbon)
  find_library(COREGRAPHICS_LIBRARY CoreGraphics)
  find_library(COREFOUNDATION_LIBRARY CoreFoundation)
  
  target_link_libraries(${PROJECT_NAME} PUBLIC
      ${COCOA_LIBRARY}
      ${IOKIT_LIBRARY}
      ${COREVIDEO_LIBRARY}
      ${OPENGL_LIBRARY}
      ${CARBON_LIBRARY}
      ${COREGRAPHICS_LIBRARY}
      ${COREFOUNDATION_LIBRARY}
  )
elseif(UNIX)
    target_compile_definitions(${PROJECT_NAME} PUBLIC LWP_PLATFORM_LINUX)
    if(LWP_USE_GLFW)
      target_compile_definitions(${PROJECT_NAME} PRIVATE LWP_USE_GLFW)
    endif()
    find_package(X11 REQUIRED)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(XRANDR REQUIRED xrandr)

    target_link_libraries(${PROJECT_NAME} PUBLIC
        ${X11_LIBRARIES}
        ${XRANDR_LIBRARIES}
    )

    target_include_directories(${PROJECT_NAME} PRIVATE
        ${XRANDR_INCLUDE_DIRS}
        ${X11_INCLUDE_DIR}
    )
endif()

# Warnings for the lib itself
if(MSVC)
  target_compile_options(${PROJECT_NAME} PRIVATE /W4 /permissive-)
else()
  target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic)
endif()

# -------- Packaging: install + Config files --------
include(CMakePackageConfigHelpers)
set(LWP_INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/Lumin)

install(TARGETS ${PROJECT_NAME}
  EXPORT LuminTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT LuminTargets
  NAMESPACE LWP::
  DESTINATION ${LWP_INSTALL_CONFIGDIR}
  FILE LuminTargets.cmake
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/LuminConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
  ${CMAKE_CURRENT_LIST_DIR}/cmake/LuminConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/LuminConfig.cmake
  INSTALL_DESTINATION ${LWP_INSTALL_CONFIGDIR}
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/LuminConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/LuminConfigVersion.cmake
  DESTINATION ${LWP_INSTALL_CONFIGDIR}
)

# -------- Examples --------
if(LWP_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()