cmake_minimum_required(VERSION 3.22)
project(BetterZoom LANGUAGES CXX)

# Debug symbols are disabled (-g0 + --strip-all).
# Built with the Android NDK toolchain file (arm64-v8a), mirroring build.sh.
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_VISIBILITY_INLINES_HIDDEN ON)

add_library(betterzoom SHARED
    src/main.cpp
    src/ZoomConfig.cpp
    src/ZoomMod.cpp
)

target_include_directories(betterzoom PRIVATE
    include
    src
    third_party
    third_party/fmt
)

target_compile_definitions(betterzoom PRIVATE FMT_HEADER_ONLY JSON_NOEXCEPTION)
target_compile_options(betterzoom PRIVATE
    -Os -g0 -w -fno-rtti -fno-exceptions
    -fno-unwind-tables -fno-asynchronous-unwind-tables
    -ffunction-sections -fdata-sections
)

add_library(preloader SHARED IMPORTED)
set_target_properties(preloader PROPERTIES
    IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/lib/libpreloader-1.5.16.so
    IMPORTED_NO_SONAME TRUE
)

target_link_options(betterzoom PRIVATE
    -Wl,-z,max-page-size=16384
    -Wl,--gc-sections
    -Wl,--strip-all
    -Wl,--exclude-libs,ALL
    -Wl,--no-as-needed
)
target_link_libraries(betterzoom PRIVATE preloader log dl m)

# The NDK toolchain adds -g to every build type; drop it from the flag sets.
string(REGEX REPLACE "(^| )-g[0-9a-z]*( |$)" " " CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
string(REGEX REPLACE "(^| )-g[0-9a-z]*( |$)" " " CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")
