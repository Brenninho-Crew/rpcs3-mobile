cmake_minimum_required(VERSION 3.28)

# --- Android Cross-Compilation Check ---
if(ANDROID OR CMAKE_SYSTEM_NAME STREQUAL "Android")
    message(STATUS "Mobile Build: Targeting Android ARM64")
    set(ANDROID_SUPPORT ON)
    add_definitions(-D__ANDROID__)
    
    # Force Mobile-friendly options
    set(USE_VULKAN ON)
    set(USE_SDL ON)
    set(USE_LIBEVDEV OFF CACHE BOOL "" FORCE)
    set(USE_GAMEMODE OFF CACHE BOOL "" FORCE)
    set(USE_DISCORD_RPC OFF CACHE BOOL "" FORCE)
    set(USE_NATIVE_INSTRUCTIONS OFF CACHE BOOL "" FORCE)
endif()

find_program(CCACHE_PATH ccache HINTS ENV PATH)
if(CCACHE_PATH)
    set(CMAKE_C_COMPILER_LAUNCHER "${CCACHE_PATH}")
    set(CMAKE_CXX_COMPILER_LAUNCHER "${CCACHE_PATH}")
endif()

project(rpcs3 LANGUAGES C CXX)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# --- Standard RPCS3 Options ---
option(WITH_LLVM "Enable usage of LLVM library" ON)
option(BUILD_LLVM "Build LLVM from git submodule" OFF)
option(USE_VULKAN "Vulkan render backend" ON)
option(USE_SDL "Enables SDL input handler" ON)

# Check for 64-bit (Required for PS3 Emulation)
if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR "RPCS3 requires a 64-bit architecture (ARM64 or x86_64).")
endif()

# --- Subdirectories ---
add_subdirectory(3rdparty)

if(NOT WIN32)
    add_compile_options(-pthread)
endif()

# Output configuration
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")

add_subdirectory(rpcs3)