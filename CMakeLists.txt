cmake_minimum_required(VERSION 3.16)
project(luasocket C)

# --- User-configurable options (similar to Makefile variables) ---
option(LUASOCKET_DEBUG "Enable luasocket debug timing" OFF)
set(LUA_VERSION "5.1" CACHE STRING "Lua version (5.1/5.2/5.3/5.4)")
option(BUILD_MIME_MODULE "Build mime module" ON)
option(BUILD_UNIX_MODULE "Build unix local-socket module" ON)
option(BUILD_SERIAL_MODULE "Build serial (device streams) module" OFF)

# Default to static library (user wanted to embed as static lib)
set(BUILD_SHARED_LIBS OFF)

# Compiler flags defaults — user may override via CMake vars or environment
if(LUASOCKET_DEBUG)
    add_compile_definitions(LUASOCKET_DEBUG)
endif()

message(STATUS "Using Lua include dir: ${LUAJIT_INCLUDE_DIR}")

# Collect sources for the core socket module
set(SOCKET_SOURCES
    src/luasocket.c
    src/timeout.c
    src/buffer.c
    src/io.c
    src/auxiliar.c
    src/compat.c
    src/options.c
    src/inet.c
    src/except.c
    src/select.c
    src/tcp.c
    src/udp.c
)

# MIME module
set(MIME_SOURCES
    src/mime.c
    src/compat.c
)

# Unix/local domain sockets module (optional)
set(UNIX_SOURCES
    src/buffer.c
    src/auxiliar.c
    src/options.c
    src/timeout.c
    src/io.c
    src/usocket.c
    src/unixstream.c
    src/unixdgram.c
    src/compat.c
    src/unix.c
)

# Serial device module (optional)
set(SERIAL_SOURCES
    src/buffer.c
    src/compat.c
    src/auxiliar.c
    src/options.c
    src/timeout.c
    src/io.c
    src/usocket.c
    src/serial.c
)

# Create static library targets
add_library(luasocket_core STATIC ${SOCKET_SOURCES})
target_include_directories(luasocket_core PRIVATE ${LUAJIT_INCLUDE_DIR})

if(BUILD_MIME_MODULE)
    add_library(luasocket_mime STATIC ${MIME_SOURCES})
    target_include_directories(luasocket_mime PRIVATE ${LUAJIT_INCLUDE_DIR})
endif()

if(BUILD_UNIX_MODULE)
    add_library(luasocket_unix STATIC ${UNIX_SOURCES})
    target_include_directories(luasocket_unix PRIVATE ${LUAJIT_INCLUDE_DIR})
endif()

if(BUILD_SERIAL_MODULE)
    add_library(luasocket_serial STATIC ${SERIAL_SOURCES})
    target_include_directories(luasocket_serial PRIVATE ${LUAJIT_INCLUDE_DIR})
endif()

# Create a combined target that includes all enabled modules
add_library(luasocket INTERFACE)
target_link_libraries(luasocket INTERFACE luasocket_core)

if(BUILD_MIME_MODULE)
    target_link_libraries(luasocket INTERFACE luasocket_mime)
endif()

if(BUILD_UNIX_MODULE)
    target_link_libraries(luasocket INTERFACE luasocket_unix)
endif()

if(BUILD_SERIAL_MODULE)
    target_link_libraries(luasocket INTERFACE luasocket_serial)
endif()

message(STATUS "luasocket targets configured successfully")