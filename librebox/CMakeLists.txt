# CMakeLists.txt (Project Root)
cmake_minimum_required(VERSION 3.20)
project(librebox LANGUAGES C CXX)
include(FetchContent)

if(NOT CMAKE_BUILD_TYPE)
  # Default to release
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build" FORCE)
endif()

# MSVC runtime (/MD)
if(MSVC)
  set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreadedDLL")
endif()

# Debug
set(CMAKE_CXX_FLAGS_DEBUG "")
set(CMAKE_C_FLAGS_DEBUG "")
if(MSVC)
  set(CMAKE_CXX_FLAGS_DEBUG "/EHsc /Od /Zi /MDd /DDEBUG /D_DEBUG")
  set(CMAKE_C_FLAGS_DEBUG "/Od /Zi /MDd /DDEBUG /D_DEBUG")
else()
  set(CMAKE_CXX_FLAGS_DEBUG "-O0 -g -Wall -Wextra -DDEBUG")
  set(CMAKE_C_FLAGS_DEBUG "-O0 -g -Wall -Wextra -DDEBUG")
endif()

# Release
set(CMAKE_CXX_FLAGS_RELEASE "")
set(CMAKE_C_FLAGS_RELEASE "")
if(MSVC)
  set(CMAKE_CXX_FLAGS_RELEASE "/EHsc /O2 /MD /DNDEBUG /DNOMINMAX")
  set(CMAKE_C_FLAGS_RELEASE "/O2 /MD /DNDEBUG")
else()
  set(CMAKE_CXX_FLAGS_RELEASE "-O2 -DNDEBUG")
  set(CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")
endif()

# Paths
set(PROJ_ROOT "${CMAKE_SOURCE_DIR}")

# ---------- Fetch Luau ----------
FetchContent_Declare(
  luau
  GIT_REPOSITORY https://github.com/luau-lang/luau.git
  GIT_TAG 0.688
)
set(LUAU_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(LUAU_BUILD_CLI OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(luau)

# ---------- Fetch Raylib ----------
FetchContent_Declare(
  raylib
  GIT_REPOSITORY https://github.com/raysan5/raylib.git
  GIT_TAG 5.5
)
FetchContent_MakeAvailable(raylib)


# ---------- Engine exe ----------
file(GLOB_RECURSE ENGINE_SOURCES
  "${PROJ_ROOT}/bootstrap/*.cpp"
  "${PROJ_ROOT}/core/*.cpp"
  "${PROJ_ROOT}/services/*.cpp"
  "${PROJ_ROOT}/subsystems/*.cpp"
)
add_executable(librebox ${ENGINE_SOURCES})

target_include_directories(librebox PRIVATE
  "${PROJ_ROOT}"
  "${luau_SOURCE_DIR}/Common/include"
  "${luau_SOURCE_DIR}/Ast/include"
  "${luau_SOURCE_DIR}/Compiler/include"
  "${luau_SOURCE_DIR}/Config/include"
  "${luau_SOURCE_DIR}/VM/include"
)

target_compile_features(librebox PRIVATE cxx_std_20)


# ---------- Linking ----------
target_link_libraries(librebox PRIVATE Luau.Ast Luau.Compiler Luau.Config Luau.VM raylib)

if(WIN32)
  target_link_libraries(librebox PRIVATE opengl32 gdi32 winmm user32 shell32)
elseif(UNIX AND NOT APPLE)
  target_link_libraries(librebox PRIVATE GL X11 pthread dl m)
endif()


# ---------- Installation (The new "Dist" step) ----------
install(TARGETS librebox
  RUNTIME DESTINATION .
)
set(CONTENT_SRC "${PROJ_ROOT}/content/dist")
if(EXISTS "${CONTENT_SRC}")
  install(DIRECTORY "${CONTENT_SRC}/"
    DESTINATION .
  )
endif()

# Add version resource on Windows
if(WIN32)
  target_sources(librebox PRIVATE "${PROJ_ROOT}/librebox.rc")
endif()

# Set final output name
set_target_properties(librebox PROPERTIES OUTPUT_NAME "LibreboxPlayer")
