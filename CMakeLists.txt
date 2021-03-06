cmake_minimum_required(VERSION 2.6)


### Meta ###

project(openlux)

set(VERSION_MAJOR 0)
set(VERSION_MINOR 2)
set(VERSION_PATCH 1)
set(VERSION_STR "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
add_definitions("-DVERSION_STR=\"${VERSION_STR}\"")


### Options ###

option(USE_IOS "Build for iOS if environment detected" ON)
option(USE_X11 "Build with X11 support" ON)


### Project variables ###

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Wno-unused-label -std=gnu99")
set(CMAKE_C_FLAGS_DEBUG "-g -O0")
set(CMAKE_C_FLAGS_RELEASE "-Ofast")

if (NOT CMAKE_BUILD_TYPE)
  message("Setting build type to Debug")
  set(CMAKE_BUILD_TYPE "Debug")
endif()


### Check system ###

# Check for X11
find_package(X11)
if (X11_FOUND AND X11_xf86vmode_FOUND)
  set(HAS_X11 on)
  set(FOUND_VIDEO_BACKEND on)
else()
  message("X11 not found")
endif()


include(CheckSymbolExists)
check_symbol_exists(clock_nanosleep "time.h" HAS_CLOCK_NANOSLEEP)
if (HAS_CLOCK_NANOSLEEP)
  add_definitions("-DOL_CMAKE_HAVE_CLOCK_NANOSLEEP=1")
endif (HAS_CLOCK_NANOSLEEP)

check_symbol_exists(nanosleep "time.h" HAS_NANOSLEEP)
if (HAS_NANOSLEEP)
  add_definitions("-DOL_CMAKE_HAVE_NANOSLEEP=1")
endif (HAS_NANOSLEEP)


# Hacky way to find if we're building for iOS (requires a toolchain with SDK_ROOT
#   pointing to the root directory of the iOS SDK)
if (EXISTS "${SDK_ROOT}/System/Library/PrivateFrameworks/IOMobileFramebuffer.framework")
  set(HAS_IOS on)
  set(FOUND_VIDEO_BACKEND on)
else()
  message("iOS SDK not found")
endif ()

if (NOT FOUND_VIDEO_BACKEND)
  message(FATAL_ERROR "Unable to find video backend")
endif (NOT FOUND_VIDEO_BACKEND)

if (HAS_IOS AND USE_IOS)

  message("Building for iOS")
  set(BUILD_IOS on)
  add_definitions("-DOL_CMAKE_USE_IOS=1")
  set(VIDEO_BACKEND_LIBS "")

  set(CMAKE_OSX_SYSROOT ${SDK_ROOT})
  include_directories(${SDK_ROOT}/usr/include)

  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -F${SDK_ROOT}/System/Library/PrivateFrameworks -framework IOKit -framework IOMobileFramebuffer")

else ()

  if (HAS_X11 AND USE_X11)
    message("Building with X11 support")
    set(BUILD_X11 on)
    add_definitions("-DOL_CMAKE_USE_X11=1")
    set(VIDEO_BACKEND_LIBS ${X11_LIBRARIES} ${X11_Xxf86vm_LIB})
  endif ()

endif ()


add_subdirectory(src)
