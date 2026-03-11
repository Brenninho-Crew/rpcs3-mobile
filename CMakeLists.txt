cmake_minimum_required(VERSION 3.28)

# --- Android Detection ---
if(ANDROID OR CMAKE_SYSTEM_NAME STREQUAL "Android")
    message(STATUS "Configuring RPCS3 for Android ARM64")
    set(ANDROID_SUPPORT ON)
    add_definitions(-D__ANDROID__)
    
    # Force built-in dependencies to avoid WolfSSL/Curl errors
    set(USE_SYSTEM_CURL OFF CACHE BOOL "" FORCE)
    set(USE_SYSTEM_ZLIB OFF CACHE BOOL "" FORCE)
    set(USE_WOLFSSL OFF CACHE BOOL "" FORCE)
    set(USE_SYSTEM_LIBUSB OFF CACHE BOOL "" FORCE)
    
    # Disable desktop-only features
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

# Requirements check
if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR "RPCS3 requires a 64-bit architecture.")
endif()

# --- Subdirectories ---
add_subdirectory(3rdparty)

if(NOT WIN32)
    add_compile_options(-pthread)
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")

add_subdirectory(rpcs3)