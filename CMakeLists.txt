cmake_minimum_required(VERSION 3.28)

# --- Android Detection & Toolchain Setup ---
if(ANDROID)
    message(STATUS "Targeting Android Platform (ARM64)")
    set(CMAKE_CXX_STANDARD 20)
    add_definitions(-D__ANDROID__ -DANDROID)
    # Android uses Vulkan and SDL for input/windowing
    set(USE_VULKAN ON)
    set(USE_SDL ON)
endif()

find_program(CCACHE_PATH ccache HINTS ENV PATH)
if(CCACHE_PATH)
  message(STATUS "Using ccache: ${CCACHE_PATH}")
  set(CMAKE_C_COMPILER_LAUNCHER "${CCACHE_PATH}")
  set(CMAKE_CXX_COMPILER_LAUNCHER "${CCACHE_PATH}")
endif()

project(rpcs3 LANGUAGES C CXX)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# --- Compiler Requirements ---
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 13)
        message(FATAL_ERROR "RPCS3 requires at least gcc-13.")
    endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    # Android NDK usually uses Clang. 
    # For mobile, we relax this slightly if using NDK, or keep for bleeding edge.
    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 17.0 AND NOT ANDROID)
        message(FATAL_ERROR "RPCS3 requires at least clang-19.0 on Desktop.")
    endif()
endif()

# --- Platform Defaults ---
if(APPLE OR WIN32 OR ANDROID)
    set(USE_SYSTEM_OPENAL_DEFAULT OFF)
else()
    set(USE_SYSTEM_OPENAL_DEFAULT ON)
endif()

# --- Mobile vs Desktop Options ---
if(ANDROID)
    option(USE_NATIVE_INSTRUCTIONS "Optimization for mobile CPU" OFF)
    option(WITH_LLVM "Enable LLVM" ON)
    option(USE_LIBEVDEV "Linux input" OFF)
    option(USE_GAMEMODE "Linux Gamemode" OFF)
    option(USE_DISCORD_RPC "Discord integration" OFF)
else()
    option(USE_NATIVE_INSTRUCTIONS "Use -march=native" ON)
    option(USE_LIBEVDEV "libevdev-based joystick support" ON)
    option(USE_GAMEMODE "Enable GameMode features" ON)
    option(USE_DISCORD_RPC "Discord rich presence integration" OFF)
endif()

option(BUILD_LLVM "Build LLVM from git submodule" OFF)
option(USE_VULKAN "Vulkan render backend" ON)
option(USE_SDL "Enables SDL input handler" ON)
option(USE_LTO "Use LTO for building" OFF) # LTO is often too heavy for mobile builds

# --- System Library Options ---
option(USE_SYSTEM_CURL "Prefer system Curl" ON)
option(USE_SYSTEM_ZLIB "Prefer system ZLIB" ON)
# (Keep other USE_SYSTEM options as needed)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/buildfiles/cmake")
include(CheckCXXCompilerFlag)

# --- Architecture Check ---
if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR "RPCS3 is 64-bit only (x64 or ARM64).")
endif()

# --- Subdirectories ---
add_subdirectory(3rdparty)

# Handle LTO conflicts
if (NOT USE_LTO)
    add_compile_options(-fno-lto)
endif()

if(NOT WIN32)
    add_compile_options(-pthread)
endif()

# --- Output Directories ---
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")

add_subdirectory(rpcs3)
