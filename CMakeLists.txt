cmake_minimum_required(VERSION 3.15)

project(AUDIOGRIDDER VERSION 1.0.0)

file(STRINGS ${CMAKE_SOURCE_DIR}/package/VERSION.num AG_VERSION)

set(CMAKE_CXX_EXTENSIONS OFF)

set(AG_VST2_SDK_DEFAULT "")
set(AG_JUCE_DEFAULT "")
if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  option(AG_MACOS_TARGET "macOS target, default is 10.8" 10.8)
  set(AG_VST2_SDK_DEFAULT "$ENV{HOME}/audio/vstsdk2.4")
  set(CMAKE_OSX_DEPLOYMENT_TARGET ${AG_MACOS_TARGET})
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
  set(CMAKE_EXPORT_COMPILE_COMMANDS TRUE)
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  set(AG_VST2_SDK_DEFAULT "z:/vstsdk2.4")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(AG_VST2_SDK_DEFAULT "")
  set(AG_JUCE_DEFAULT "/usr/share/juce")
endif()

# set(AG_VST2_SDK ${AG_VST2_SDK_DEFAULT} CACHE STRING "VST2 SDK Folder")

set(JUCE_DIR ${AG_JUCE_DEFAULT} CACHE STRING "JUCE Directory")

add_subdirectory(${JUCE_DIR} JUCE)

if(USE_VST STREQUAL "1")
  juce_set_vst2_sdk_path(${AG_VST2_SDK})
# else()
#  list(REMOVE_ITEM FormatsToBuild VST)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

# does not work on Manjaro system with libs installed in /usr (which is quite standard?!)
# find_package(FFmpeg REQUIRED)

# https://stackoverflow.com/questions/50760024/cmake-configuration-for-ffmpeg-in-c-project
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBAV REQUIRED IMPORTED_TARGET
    libavdevice
    libavfilter
    libavformat
    libavcodec
    libswresample
    libswscale
    libavutil
)

# find_package(WebP REQUIRED)
pkg_check_modules(LIBWP REQUIRED IMPORTED_TARGET
  libwebp
  libwebpdecoder
  libwebpdemux
  libwebpmux
)

find_package(Boost 1.70.0 REQUIRED)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/Common/Source)
include_directories(${FFMPEG_INCLUDE_DIRS})
include_directories(${WEBP_INCLUDE_DIRS})
include_directories(${Boost_INCLUDE_DIRS})

# link_directories(${FFMPEG_LIBRARY_DIRS})
link_directories(PkgConfig::LIBAV)
# target_link_libraries(${PROJECT_NAME}
#  PkgConfig::LIBAV
#)
# link_directories(${WEBP_LIBRARY_DIRS})
link_directories(PkgConfig::LIBWP)

aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/Common/Source AG_SOURCES_COMMON)
list(REMOVE_ITEM AG_SOURCES_COMMON "${CMAKE_CURRENT_SOURCE_DIR}/Common/Source/TraceReader.cpp")

option(AG_WITH_PLUGIN "Enable Plugin build." on)
option(AG_WITH_SERVER "Enable Server build." on)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(AG_WITH_SERVER off)
endif()

if(AG_WITH_PLUGIN)
  message(STATUS "Plugin enabled. Use -DAG_WITH_PLUGIN=OFF to deactivate building the plugins.")
  add_subdirectory(Plugin)
else()
  message(STATUS "Plugin disabled.")
endif()

if(AG_WITH_SERVER)
  message(STATUS "Server enabled. Use -DAG_WITH_SERVER=OFF to deactivate building the server.")
  add_subdirectory(Server)
else()
  message(STATUS "Server disabled.")
endif()
